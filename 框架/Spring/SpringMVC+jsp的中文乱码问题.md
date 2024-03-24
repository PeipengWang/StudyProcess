# SpringMVC+jsp中文乱码问题解决
1、首先要添加中文过滤器
```
<filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
org.springframework.web.filter.CharacterEncodingFilter 这个字符编码过滤器通常应该位于 org.springframework.web.servlet.DispatcherServlet 的前面，确保在请求进入控制器之前对字符编码进行正确的设置
2、设置响应格式
 @GetMapping(value = "/getTable", produces = "text/html; charset=UTF-8")
 produces 属性是 @GetMapping 注解的一个选项属性，用于指定处理方法可以生产的媒体类型（即响应的内容类型）。

 3、如果用的jsp则需要设置page的contentType
```
 <%@ page contentType="text/html; charset=UTF-8" %>
```
其他前端页面也有各自的，需要注意
4、如果还不行可以自设置tomcat
 修改conf目录下的server.xml
```
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           URIEncoding="UTF-8" />
```
上面的示例中，URIEncoding属性被设置为UTF-8，这会影响URL中的字符编码处理。谨慎修改server.xml文件