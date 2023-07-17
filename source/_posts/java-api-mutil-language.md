---
title: 接口层面集成翻译插件实现返回结果多语言的方案
tags: [java, 多语言, api]
categories: [java]
index_img: /img/index_img/api-multi-language.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

### 背景及设计思路

> 在做接口开发的时候给客户端的响应需要根据不同的语言返回对应的结果，主要核心内容有两点：

- 返回结果做多语言转换
- 接口进行统一处理

### 具体实现

> 1. 引入腾讯云sdk

```java
<!--	腾讯云sdk-->
  <dependency>
   <groupId>com.tencentcloudapi</groupId>
   <artifactId>tencentcloud-sdk-java</artifactId>
   <version>3.0.29</version>
  </dependency>
```

> 2. 实现多语言翻译工具类

登录腾讯云平台 -> 搜索机器翻译 -> 资源包 -> 创建资源-> 点击用户头像-> 访问管理 -> 获取密钥

代码：

```java
package org.jeecg.modules.dzcloud.utils;

import com.tencentcloudapi.common.Credential;
import com.tencentcloudapi.common.exception.TencentCloudSDKException;
import com.tencentcloudapi.common.profile.ClientProfile;
import com.tencentcloudapi.common.profile.HttpProfile;
import com.tencentcloudapi.tmt.v20180321.TmtClient;
import com.tencentcloudapi.tmt.v20180321.models.TextTranslateRequest;
import com.tencentcloudapi.tmt.v20180321.models.TextTranslateResponse;

/**
 * @Description
 * @Author Jay
 * @Date 2023/7/14 16:47
 **/
public class TranslateUtils {

    /**
     *
     * @param sourceText 带翻译文本
     * @param source 原语言
     * @param target 目标语言
     * @return
     */
    public static String textTranslate(String sourceText, String source, String target) {
        try {
            // 设置腾讯云 API 密钥
            Credential cred = new Credential("AKIDPze6MTZTBpOsRB0f8RcaSYQH0X4XwALS", "ELGAvdPTehjdnupdEdAXlDdQMEAzazSq");

            // 创建 HTTP 参数配置
            HttpProfile httpProfile = new HttpProfile();
            httpProfile.setEndpoint("tmt.tencentcloudapi.com"); // 设置 API 地域

            // 创建客户端配置
            ClientProfile clientProfile = new ClientProfile();
            clientProfile.setHttpProfile(httpProfile);

            // 创建翻译客户端
            TmtClient client = new TmtClient(cred, "ap-guangzhou", clientProfile);

            // 创建翻译请求
            TextTranslateRequest request = new TextTranslateRequest();
            request.setSourceText(sourceText);
            request.setSource(source);
            request.setTarget(target);
            request.setProjectId(0); // 可选的项目 ID

            // 发送翻译请求
            TextTranslateResponse response = client.TextTranslate(request);

            // 返回翻译结果
            return response.getTargetText();
        } catch (TencentCloudSDKException e) {
            e.printStackTrace();
            return e.getMessage();
        }

    }

}
```

> 3. aop切面统一处理,实现返回结果拦截翻译再返回,直接上代码

```java
import lombok.AllArgsConstructor;
import org.apache.commons.lang3.StringUtils;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springblade.core.tool.api.R;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
 
import javax.servlet.http.HttpServletRequest;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Map;
import java.util.Objects;
import java.util.concurrent.ConcurrentHashMap;
 
/**
 * 对请求参数进行拦截修改
 * @author
 */
@Aspect
@Component
@AllArgsConstructor
@ConditionalOnProperty(prefix = "lang",name = "open",havingValue = "true")
public class LanguageAspect {
 
 private String objPreFix = "R";
 
 @Pointcut("execution(* com.rksj.controller.*.*(..))")
 public void annotationLangCut(){};
 
 /**
  * 定义一个并发性Map用来存放信息
  */
 Map<String, String> tmpMap = new ConcurrentHashMap<>();
 
 /**
  * 在构造函数中进行初始化，后面可从配置文件中初始化
  */
 public LanguageAspect(){
  //初始化 KEY为简体  VALUE为  简体###繁体###英文
  loadSysCfg();
 
 }
 
 /**
  * 从配置文件加载
  */
 private void loadSysCfg(){
  try{
   //加载配置文件
   Resource resource = new ClassPathResource("language");
   InputStream is = resource.getInputStream();
   InputStreamReader inputStreamReader = new InputStreamReader(is);
   BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
   String line = "";
   while( (line = bufferedReader.readLine()) != null ){
    //不为空且以#号开头的去掉，注释部分
    if(StringUtils.isNotEmpty(line) && !line.startsWith("#")){
     String[] lines = line.split("###");
     tmpMap.put(lines[0], line);
    }
   }
  }catch (Exception e){
   e.printStackTrace();
  }
 }
 
 
 
 /**
  * 拦截controller层返回的结果，修改msg字段
  * @param point
  * @param obj
  */
 @AfterReturning(pointcut="annotationLangCut()",returning="obj")
 public void around(JoinPoint point, Object obj)  {
  Map<String, String> map = new HashMap<String, String>();
  Object backObj = obj;
  try{
   RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
   //从获取RequestAttributes中获取HttpServletRequest的信息
   HttpServletRequest request = (HttpServletRequest) requestAttributes.resolveReference(RequestAttributes.REFERENCE_REQUEST);
   String bladeLang = request.getHeader("x-language");
   String targetTxt = TranslateUtils.textTranslate(obi.JSON.toJSONString(), "auto" ,bladeLang);
   obj = JSON.parseObject(targetTxt);
   }
  }catch (Exception e){
   e.printStackTrace();
   //重新赋值给obj，防止try中途修改原始的值
   obj = backObj;
  }
```
>
> 4. 调用及返回结果示例

调用：

```java
header: {
    "X-Lang-Target": "ja", //目标语言，日语
}
params: {
    "text": "你好",
}
```

返回:

```java
{
    "success": true,
    "message": "こんにちは",
    "code": 0,
    "result": null,
    "timestamp": 1689557575076
}
```