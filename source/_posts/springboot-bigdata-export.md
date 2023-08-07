---
title: SpringBoot 实现 MySQL 百万级数据量导出并避免 OOM 的解决方案
tags: [SpringBoot, MySQL, 海量数据]
categories: [剑指 Offer II]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/bigDataExport.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

### 引言
>
> 数据导出在项目开发中非常常见，一般的处理流程就是从数据库读取数据，生成excel等格式数据，通过流形式输出给前端。但百万级数据量的数据导出其实用到的并不多，主要在于业务上可能也没有必要性。今天主要讨论下从技术的角度出发，怎样实现百万级数据量导出并且不造成OOM。
>
### 思路
>
> 由于数据量很大，全量加载必然不行，所以我们采用分批加载，而Mysql本身支持Stream查询，我们可以通过Stream流获取数据，然后将数据逐条刷入到文件中，每次刷入文件后再从内存中移除这条数据，从而避免OOM。

由于采用了数据逐条刷入文件，而且数据量达到百万级，所以文件格式就不要采用excel了，excel2007最大才支持104万行的数据,所以我们选择使用csv格式。

### 具体实现

- jpa实现

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
