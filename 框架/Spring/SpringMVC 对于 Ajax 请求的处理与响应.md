   在浏览器中发出 Ajax 请求后一般会返回一个 json 数据，前端会根据 这个返回的 json 数据来进一步的处理，本文主要关注于 SpringMVC 会获得 Ajax 请求后，将响应数据转化为 json 形式完成前后端的交互。 
   首先，需要了解 SpringMVC 是怎样将数据转化为 json 形式的， 这里要用到一个注解 @ResponseBody。
   这个注解的作用可以将一个返回的数据不是作为解析页面的形式返回， 而是以一个数据的形式返回，例如
   在前端定义一个超链接
   

```html
<a th:href="@{/testResponseBody}">测试ResponseBody</a><br>
```
对应的处理方法，如果不加 @ResponseBody ：

```java
   @RequestMapping(value = "testResponseBody")
    public String testHttpRequestBody(){
       return "success";
    }
```
这时候这个返回的 success 会通过前端页面解析器，如 themleaf 解析器，解析出名字例如为 success.html 的页面。
但是如果加上 @ResponseBody 的注解

```java
    @RequestMapping(value = "testResponseBody")
    @ResponseBody
    public String testHttpRequestBody(){
       return "success";
    }
```
他会单纯的返回 success 在页面上。
但是如果返回一个实体类，省略构造方式，set 和 get 方法如下所示：

```java
public class User {
   private String username;
   private String password;
   private String sex;
   private String email;
   }
```
处理方法变为：

```java
    @RequestMapping(value = "testResponseBody")
    @ResponseBody
    public User testHttpRequestBody(){
        User user = new User("admin","123456","女","www.@com");
       return user;
    }
```
这时，发送请求回收到一个500错误，如下所示（主要信息）：

```html
类型 异常报告
消息 No converter found for return value of type: class com.wpp.pojo.User
描述 服务器遇到一个意外的情况，阻止它完成请求。
例外情况
org.springframework.http.converter.HttpMessageNotWritableException: No converter found for return value of type: class com.wpp.pojo.User
org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor.writeWithMessageConverters(AbstractMessageConverterMethodProcessor.java:220)
```
问题原因：缺少一个转化器将数据转化成可识别的类型，这个转化器为 json 转化器，因此，在 pom.xml 引入一个即可

```xml
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.1</version>
        </dependency>
```
这时，再次发送请求会得到
