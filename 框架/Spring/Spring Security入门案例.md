Spring Security 的前身是 Acegi Security ，是 Spring 项目组中用来提供安全认证服务的框架。
安全包括两个主要操作。
“认证”，主题一般式指用户，设备或可以在你系 统中执行动作的其他系,通俗说是登陆。
统。
“授权”，权限管理。
**快速入门案例：**
配置步骤：
①，导入依赖

```c
<dependencies>
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-web</artifactId>
<version>5.0.1.RELEASE</version>
</dependency>
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-config</artifactId>
<version>5.0.1.RELEASE</version>
</dependency>
</dependencies>
```
②.web.xml配置filter

```c
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>classpath:spring-security.xml</param-value>
</context-param>
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<filter>
<filter-name>springSecurityFilterChain</filter-name>
<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
<filter-name>springSecurityFilterChain</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>
```
③.Spring Security核心文件配置

```c
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:security="http://www.springframework.org/schema/security"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/security
http://www.springframework.org/schema/security/spring-security.xsd">
<security:http auto-config="true" use-expressions="false">
<!-- intercept-url定义一个过滤规则 pattern表示对哪些url进行权限控制，ccess属性表示在请求对应
的URL时需要什么权限，
默认配置时它应该是一个以逗号分隔的角色列表，请求的用户只需拥有其中的一个角色就能成功访问对应
的URL -->
<security:intercept-url pattern="/**" access="ROLE_USER" />
<!-- auto-config配置后，不需要在配置下面信息 <security:form-login /> 定义登录表单信息
<security:http-basic
/> <security:logout /> -->
</security:http>
<security:authentication-manager>
<security:authentication-provider>
<security:user-service>
<security:user name="user" password="{noop}user"
authorities="ROLE_USER" />
<security:user name="admin" password="{noop}admin"
authorities="ROLE_ADMIN" />
</security:user-service>
</security:authentication-provider>
</security:authentication-manager>
</beans>
```
文件整体结构
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515231807204.png)
输出效果：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515231829124.png)
默认登陆
User：user
Password:user
输出：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515231941115.png)
若登陆失败：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515232011170.png)
**进一步改进**
①.登陆界面：login.html

```c
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>login.html</title>
</head>
<body>
<form action="login" method="post">
    <table>
        <tr>
            <td>姓名：</td>
            <td><input type="text" name="username" /></td>
        </tr>
        <tr>
            <td>密码：</td>
            <td><input type="password" name="password" /></td>
        </tr>
        <tr>
            <td colspan="2" align="center"><input type="submit" value="登录" />
                <input type="reset" value="重置" /></td>
        </tr>
    </table>
</form>
</body>
</html>
```
②.登陆成功界面:success.html

```c
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
success.html
</body>
</html>
```
③.登陆失败界面：failer.html

```c
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
success.html
</body>
</html>
```
④.配置spring-security.xml

```c
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/security
http://www.springframework.org/schema/security/spring-security.xsd">

    <!-- 配置不过滤的资源（静态资源及登录相关） -->
    <security:http security="none" pattern="/login.html" />
    <security:http security="none" pattern="/failer.html" />
    <security:http auto-config="true" use-expressions="false" >
        <!-- 配置资料连接，表示任意路径都需要ROLE_USER权限 -->
        <security:intercept-url pattern="/**" access="ROLE_USER" />
        <!-- 自定义登陆页面，login-page 自定义登陆页面 authentication-failure-url 用户权限校验失败之后才会跳转到这个页面，
        如果数据库中没有这个用户则不会跳转到这个页面。
            default-target-url 登陆成功后跳转的页面。 注：登陆页面用户名固定 username，密码 password，action:login -->
        <security:form-login login-page="/login.html"
                             login-processing-url="/login" username-parameter="username"
                             password-parameter="password" authentication-failure-url="/failer.html"
                             default-target-url="/success.html" authentication-success-forward-url="/success.html"
        />

        <!-- 关闭CSRF,默认是开启的 -->
        <security:csrf disabled="true" />
    </security:http>
    <security:authentication-manager>
        <security:authentication-provider>
            <security:user-service>
                <security:user name="user" password="{noop}user"
                               authorities="ROLE_USER" />
                <security:user name="admin" password="{noop}admin"
                               authorities="ROLE_ADMIN" />
            </security:user-service>
        </security:authentication-provider>
    </security:authentication-manager>
</beans>
```
**输出**
文件结构
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515232654807.png)
登陆：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515232745687.png)
登陆成功
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020051523281180.png)
登陆失败：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200515232844649.png)
重点总结：在配置过程中，主要注意spring-security.xml的配置

```
<!-- 配置不过滤的资源（静态资源及登录相关） -->
    <security:http security="none" pattern="/login.html" />
    <security:http security="none" pattern="/failer.html" />
    <security:http auto-config="true" use-expressions="false" >
``
其中，security="none"表示login.html，与failer.html是不需要权限就可以进入的。

```
 <!-- 配置资料连接，表示任意路径都需要ROLE_USER权限 -->
        <security:intercept-url pattern="/**" access="ROLE_USER" />
        <security:form-login login-page="/login.html"
                             login-processing-url="/login" username-parameter="username"
                             password-parameter="password" authentication-failure-url="/failer.html"
                             default-target-url="/success.html" authentication-success-forward-url="/success.html"
        />
```
上面代码含义是：进入login.html之后，对于成功登陆的的用户获得ROLE_USER权限，成功进入success.html，失败进入failer.html

```
    <!-- 关闭CSRF,默认是开启的 -->
        <security:csrf disabled="true" />
    </security:http>
    <security:authentication-manager>
        <security:authentication-provider>
            <security:user-service>
                <security:user name="user" password="{noop}user"
                               authorities="ROLE_USER" />
                <security:user name="admin" password="{noop}admin"
                               authorities="ROLE_ADMIN" />
            </security:user-service>
        </security:authentication-provider>
    </security:authentication-manager>
```
上面代码通过账户和密码分别配置管理权限，成功登陆user的用户获得ROLE_USER权限，成功登陆admin的用户获得ROLE_ADMIN权限。


        

