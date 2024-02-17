---
title: SpringBoot 实现 MySQL 百万级数据量导出并避免 OOM 的解决方案
tags: [SpringBoot, MySQL, 海量数据]
categories: [java]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/bigDataExport.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

### 1.引言
>
> 数据导出在项目开发中非常常见，一般的处理流程就是从数据库读取数据，生成excel等格式数据，通过流形式输出给前端。但百万级数据量的数据导出其实用到的并不多，主要在于业务上可能也没有必要性。今天主要讨论下从技术的角度出发，怎样实现百万级数据量导出并且不造成OOM。
>
### 2.思路
>
> 由于数据量很大，全量加载必然不行，所以我们采用分批加载，而Mysql本身支持Stream查询，我们可以通过Stream流获取数据，然后将数据逐条刷入到文件中，每次刷入文件后再从内存中移除这条数据，从而避免OOM。

由于采用了数据逐条刷入文件，而且数据量达到百万级，所以文件格式就不要采用excel了，excel2007最大才支持104万行的数据,所以我们选择使用csv格式。

### 3.具体实现

#### 3.1 jpa实现

>核心注解如下，需要加入到具体的Repository之上。方法的返回类型定义成Stream。Integer.MIN_VALUE告诉jdbc driver逐条返回数据。

```java
@QueryHints(value = @QueryHint(name = HINT_FETCH_SIZE, value = "" + Integer.MIN_VALUE))
@Query(value = "select t from Todo t")
Stream<Todo> streamAll();
```

> 此外还需要在Stream处理数据的方法之上添加@Transactional(readOnly = true)，保证事物是只读的。

>同时需要注入javax.persistence.EntityManager，通过detach从内存中移除已经使用后的对象。

```java
@RequestMapping(value = "/todos.csv", method = RequestMethod.GET)
@Transactional(readOnly = true)
public void exportTodosCSV(HttpServletResponse response) {
 response.addHeader("Content-Type", "application/csv");
 response.addHeader("Content-Disposition", "attachment; filename=todos.csv");
 response.setCharacterEncoding("UTF-8");
 try(Stream<Todo> todoStream = todoRepository.streamAll()) {
  PrintWriter out = response.getWriter();
  todoStream.forEach(rethrowConsumer(todo -> {
   String line = todoToCSV(todo);
   out.write(line);
   out.write("\n");
   entityManager.detach(todo);
  }));
  out.flush();
 } catch (IOException e) {
  log.info("Exception occurred " + e.getMessage(), e);
  throw new RuntimeException("Exception occurred while exporting results", e);
 }
}
```

#### 3.2 并发查询、读写分离

- 并发查询

> 根据数据总量计算出需并发查询的次数，线程池执行并发查询放入阻塞队列。

```java
    /**
     * 并发查询数据库
     * @param queue   存放数据队列
     * @param loopNum 查询次数
     */
    public void executeTask(ArrayBlockingQueue<List<Map<String, String>>> queue, int loopNum) {
        //loopNum并发查询次数
        for (int i = 1; i <= loopNum; i++) {
            //并发查询后放入队列中
            int finalI = i;
            executor.execute(() -> {
            long s = System.currentTimeMillis();
            List<Map<String, String>> dataList = projectService.myPage(finalI, PAGE_NUM);
            try {
                queue.put(dataList);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            });
            log.info("开始查询第{}条开始的{}条记录", i * PAGE_NUM, PAGE_NUM);
        }

    }
```

- 读写分离

> 大数据量使用`ExcelBigWriter`,从阻塞队列中不断读取数据写入writer，设置AtomicInteger为写入次数（即并发查询次数），
> 每读取一次减一，直到atomicInteger为0跳出循环。

```java
    /*** 
     * @Description excel导出，将并发查询数据写入filePath文件
     * @Param filePath
     * @Return void
     **/
    public void export(String filePath) {
        long start = System.currentTimeMillis();
        OutputStream out = null;
        ExcelWriter writer = ExcelUtil.getBigWriter();
        setStyle(writer);
        try {
            out = new FileOutputStream(filePath);
            //获取总数
            long count = projectService.count();
            //计算开启的线程数
            int loopNum = new Double(Math.ceil((double) count / PAGE_NUM)).intValue();
            log.info("多线程查询，总数：{}，开启线程数：{}", count, loopNum);
            //队列存放数据库中查询导的数据
            ArrayBlockingQueue<List<Map<String, String>>> queue = new ArrayBlockingQueue<>(loopNum, true);
            //当数据为空时跳出写入循环
            AtomicInteger atomicInteger = new AtomicInteger(loopNum);
            //并发查询数据
            executeTask(queue, loopNum);
            List<Map<String, String>> dataList = null;
            //读写分离式写入
            while ((dataList = queue.poll(PAGE_SEARCH_TIMEOUT_SECONDS, TimeUnit.SECONDS)) != null) {
                writer.write(dataList);
                //跳出循环
                if ( atomicInteger.decrementAndGet() == 0) {
                    break;
                }
            }
            long end = System.currentTimeMillis();
            log.info("导出耗时：" + (end - start));
        } catch (Exception e) {
            log.debug("文件导出报错，{}", e.getMessage());
        } finally {
            if (out != null) {
                writer.flush(out, true);
                writer.close();
            }
        }
    }
```

***附源码***

```java
/**
 * @Description
 * @Date 2023/11/29 16:18
 **/
@Service
public class ExcelService {
    private final static Logger log = LoggerFactory.getLogger(ExcelService.class);
    @Resource
    private ThreadPoolExecutor executor;

    @Resource
    private IJoaProjectService projectService;

    /**
     * 每个线程查询的页数
     */
    private static final int PAGE_NUM = 80000;

    /**
     * 阻塞队列获取数据超时时间
     */
    private static final Integer PAGE_SEARCH_TIMEOUT_SECONDS = 60;


    /***
     * @Description excel导出，将并发查询数据写入filePath文件
     * @Param filePath
     * @Return void
     **/
    public void export(String filePath) {
        long start = System.currentTimeMillis();
        OutputStream out = null;
        ExcelWriter writer = ExcelUtil.getBigWriter();
        setStyle(writer);
        try {
            out = new FileOutputStream(filePath);
            //获取总数
            long count = projectService.count();
            //计算开启的线程数
            int loopNum = new Double(Math.ceil((double) count / PAGE_NUM)).intValue();
            log.info("多线程查询，总数：{}，开启线程数：{}", count, loopNum);
            //队列存放数据库中查询导的数据
            ArrayBlockingQueue<List<Map<String, String>>> queue = new ArrayBlockingQueue<>(loopNum, true);
            //当数据为空时跳出写入循环
            AtomicInteger atomicInteger = new AtomicInteger(loopNum);
            //并发查询数据
            executeTask(queue, loopNum);
            List<Map<String, String>> dataList = null;
            //读写分离式写入
            while ((dataList = queue.poll(PAGE_SEARCH_TIMEOUT_SECONDS, TimeUnit.SECONDS)) != null) {
                writer.write(dataList);
                //跳出循环
                if ( atomicInteger.decrementAndGet() == 0) {
                    break;
                }
            }
            long end = System.currentTimeMillis();
            log.info("导出耗时：" + (end - start));
        } catch (Exception e) {
            log.debug("文件导出报错，{}", e.getMessage());
        } finally {
            if (out != null) {
                writer.flush(out, true);
                writer.close();
            }
        }
    }


    /**
     * 并发查询数据库
     *
     * @param queue   存放数据队列
     * @param loopNum 查询次数
     */

    public void executeTask(ArrayBlockingQueue<List<Map<String, String>>> queue, int loopNum) {
        //loopNum并发查询次数
        for (int i = 1; i <= loopNum; i++) {
            //并发查询后放入队列中
            int finalI = i;
            executor.execute(() -> {
            long s = System.currentTimeMillis();
            List<Map<String, String>> dataList = projectService.myPage(finalI, PAGE_NUM);
            try {
                queue.put(dataList);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            });
            log.info("开始查询第{}条开始的{}条记录", i * PAGE_NUM, PAGE_NUM);
        }

    }

    //设置表格样式
    private void setStyle(ExcelWriter writer) {
        int columnSize = 26;
        CellStyle cellStyle = writer.getCellStyle();
        cellStyle.setBorderTop(BorderStyle.THIN);
        cellStyle.setBorderBottom(BorderStyle.THIN);
        cellStyle.setBorderLeft(BorderStyle.THIN);
        cellStyle.setBorderRight(BorderStyle.THIN);
        cellStyle.setAlignment(HorizontalAlignment.LEFT);
        for (int i = 0; i < columnSize; i++) {
            if (i == 3 || i == 4 || i == 14 || i == 22 || i == 25) {
                writer.setColumnWidth(i, 24);
            } else if (i == 18 || i == 0) {
                writer.setColumnWidth(i, 48);
            } else {
                writer.setColumnWidth(i, 12);
            }
            writer.setColumnStyle(i, cellStyle);
        }
    }
}
```
