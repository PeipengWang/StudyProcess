Spring Boot项目出现问题：
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.
Tue Jun 07 11:39:11 CST 2022
There was an unexpected error (type=Not Found, status=404).

主要原因是
主类（@SpringBootApplication接口注释的启动位置）应该与运行的所有包处在同一级别。
因为Spring 将仅扫描主类包下的类。
com
   +- APP
         +- Application.java  <--- 你的主类应该位于这个地方
         |
         +- model
         |   +- user.java
         +- controller
             +- UserController.java

