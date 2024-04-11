 **转发与重定向的区别**


转发是实际上是服务器内部的一次请求，重定向是客户端的两次请求。基本表现为转发时候浏览器地址不会发生改变，而重定向会改变。由此导致对应 Request 域来说，转发能够在内共享数据，而重定向则不能。当然，转发只能跳转服务器内部的URL指令，重定向能够跳转服务器外部URL。两者各有用途，本文主要简单罗列在 Spring MVC 中的基本使用。


直接将 Spring MVC 的转发与重定向代码复制如下：

```java
@Controller
@RequestMapping("/testView")
public class VieController {
    @RequestMapping("/testThymeleafView")
    public String thymeleafView(){
        return "first";
    }
    @RequestMapping("/testForward")
    public String forwardView(){
        return "forward:/testView/testThymeleafView";
    }
    @RequestMapping("/testRedirect")
    public String redirectView(){
        return "redirect:/testView//testThymeleafView";
    }
}
```
**浏览器代码地址**
其中第一个请求 http://localhost:8080/SpringMVC/testView/testForward 为转发和重定向的页面，这里利用了 ThymeleafView 技术，转发的页面为 first.html 。
/testForward 请求为转发页面。 /testRedirect 为重定向页面。
首先，观察浏览器的不同点，
分别在输入：
http://localhost:8080/SpringMVC/testView/testRedirect
http://localhost:8080/SpringMVC/testView/testForward
浏览器会显示：
http://localhost:8080/SpringMVC/testView//testThymeleafView
http://localhost:8080/SpringMVC/testView/testForward
浏览器地址会在重定向的时候再次发送一个新的URL，在转发时则不会改变。

**request 于内容变化**
    为了更好的测试，将后端代码改为:

```java
@Controller
@RequestMapping("/testView")
public class VieController {
    @RequestMapping("/testThymeleafView")
    public String thymeleafView(HttpServletRequest request){
        return "first";
    }
    @RequestMapping("/testForward")
    public String forwardView(HttpServletRequest request){
        request.setAttribute("test","testForward");
        return "forward:/testView/testThymeleafView";
    }
    @RequestMapping("/testRedirect")
    public String redirectView(HttpServletRequest request){
        request.setAttribute("test","testRedirect");
        return "redirect:/testView/testThymeleafView";
    }
}
```
这里主要分别修改 Request 域的数据，其中 键值为 test ，对于转发与重定向分别写入，对前端页面进行展示最终 Request 域中 test 的 value 值。
代码如下：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.themleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
测试结果：
<p th:text="${test}"></p>
</body>
</html>
```
浏览器输入：http://localhost:8080/SpringMVC/testView/testForward
前端显示为

```html
测试结果：
testForward
```
由此可知在转发状态下，转发的页面能够进行数据共享的。

浏览器输入：http://localhost:8080/SpringMVC/testView/testRedirect
测试结果

```html
测试结果：
```
结果为空可知，重定向无法共享 Request 域的数据。
当然，Application，Session 这两个更大范围的域在转发和重定向都是可以共享数据的。

在 Spring MVC 中，无论是直接到页面还是转发重定向，都会经过SpringMVC中的视图解析器的View接口，视图的作用渲染数据，将模型Model中的数据展示给用户。SpringMVC视图的种类很多，默认有转发视图和重定向视图。若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView。
在代码中，ThymeleafView当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置
的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转。
SpringMVC中默认的转发视图是InternalResourceView，SpringMVC中创建转发视图的情况：当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转。
SpringMVC中默认的重定向视图是RedirectView，当控制器方法中所设置的视图名称以"redirect:"为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最终路径通过重定向的方式实现跳转。

Spring MVC 中确定哪个接口的源码方法如下：

```java
View view;
if (viewName != null) {
      view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);
      if (view == null) {
            throw new ServletException("Could not resolve view with name '" + mv.getViewName() + "' in servlet with name '" + this.getServletName() + "'");
     }
    } else {
       view = mv.getView();
       if (view == null) {
       throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a View object in servlet with name '" + this.getServletName() + "'");
   }}
```
