  在 Servlet 中有格HttpServletRequest 的方法可以使使我们可以轻松获取请求数据， 包括请求头、请求体。而在 SpringMVC 中一方面可以利用 Servlet 的 API 来获取请求数据；另一方面也提供了注解和实体类的方式来更加方便的获取。话不多说，直接上代码。
  其中，前端代码为  themleaf 引擎构造出来，分别构造出两个输入用户数据的文本框，在后端进行分别测试 @RequestBody 与RequestEntity  
  

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.themleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form th:action="@{/testHttpRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    爱好：<input type="checkbox" name="hobby" value="a">a
    <input type="checkbox" name="hobby" value="b">b
    <input type="checkbox" name="hobby" value="c">c<br>
    <input type="submit" value="登录">
</form>
<form th:action="@{/testHttpRequestEntity}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    爱好：<input type="checkbox" name="hobby" value="a">a
    <input type="checkbox" name="hobby" value="b">b
    <input type="checkbox" name="hobby" value="c">c<br>
    <input type="submit" value="登录">
</form>
</body>
</html>
```
后端代码

```java
package com.wpp.controller;

import org.springframework.http.RequestEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;


@Controller
@RequestMapping(value = "/")
public class HttpController {
    @RequestMapping(value = "testHttpRequestBody")
    public String testHttpRequestBody(@RequestBody  String request){
        System.out.println(request);
        return "success";
    }
    @RequestMapping(value = "testHttpRequestEntity")
    public String testHttpRequestBody(RequestEntity<String> requestEntity){
        System.out.println("body:"+requestEntity.getBody());
        System.out.println("header:"+requestEntity.getHeaders());
        System.out.println("url:"+requestEntity.getUrl());
        return "success";
    }
}

```

 分别展示请求数据
 展示 RequestBody 为
 

```xml
username=admin&password=123456&hobby=a&hobby=b&hobby=c
```
展示 RequestEntity  实体类为
```xml
body:username=admin&password=123456&hobby=a&hobby=b&hobby=c
header:[host:"localhost:8080", connection:"keep-alive", content-length:"54", cache-control:"max-age=0", upgrade-insecure-（。。。。省略）
url:http://localhost:8080/SpringMVC/testHttpRequestEntity
```
会发现 RequestBody 仅仅是对于请求体的数据。 RequestEntity 更为全面 url、请求体、请求体、请求方法等都能获得。
对于使用方法上，@RequestBody 是注解的方式，RequestEntity 则是一个实体类。