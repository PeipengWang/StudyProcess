# 前端到后端
## 1、占位符
原始方式：/deleteUser?id=1
rest方式：/deleteUser/1
SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服务器中，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参

```html
<a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
```

```java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id,
@PathVariable("username") String username){ System.out.println("id:"+id+",username:"+username);
return "success"; }//最终输出的内容为-->id:1,username:admin
```
## 2、通过ServletAPI获取
将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象
直接构造指令

```java
@RequestMapping("/test")
public String testParam(HttpServletRequest request)
{ 
String username = request.getParameter("username"); 
String password = request.getParameter("password"); 
System.out.println("username:"+username+",password:"+password);
 return "success"; }
```

```html
<a th:href="@{/test(username="wpp",password="1234"}">测试路径servlet获取参数>/testRest</a><br>
```
占位符方式与这种方式不能一起使用
## 3、通过控制器方法的形参获取请求参数
在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参

```java
    @RequestMapping(value = "first2")
    public String first2(String username,String password,String hobby){
        System.out.println(username+"\n"+password+"\n"+hobby);
        return "first";
    }
```

```html
<form th:action="@{/first2}" method="get">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    爱好：<input type="checkbox" name="hobby" value="a">a
    <input type="checkbox" name="hobby" value="b">b
    <input type="checkbox" name="hobby" value="c">c<br>
    <input type="submit" value="登录">
</form>
```
登录发送的请求

```java
http://localhost:8080/SpringMVC/first2?username=admin&password=12&hobby=a&hobby=b
```
输出：

```bash
admin
12
a,b
```
注意：
发送页面会对密码明文显示，可以将前端发送方法改为post；
String hobby 也可以根据实际为字符串数组的方式变为 String[] hobby；
若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中设置字符串；
数组或者字符串类型的形参接收此请求参数；
若使用字符串数组类型的形参，此参数的数组中包含了每一个数据；
若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果。
## 4、@RequestParam、@RequestHeader、@CookieValue
@RequestParam是将请求参数和控制器方法的形参创建映射关系
@RequestParam注解一共有三个属性：
value：指定为形参赋值的请求参数的参数名
required：设置是否必须传输此请求参数，默认值为true
若设置为 true 时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置
defaultValue 属性，则页面报错400：Required String parameter 'xxx' is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null
defaultValue ：不管required属性值

```java
    @RequestMapping(value = "first3")
    public String first3(@RequestParam(value = "username", required = false, defaultValue = "hehe") String username,
                         String password,
                         String[] hobby,
                         @RequestHeader(value = "Host", required = false, defaultValue = "header") String header,
                         @CookieValue(value = "JSESSIONID", required = false, defaultValue = "default_session") String Jsessionid){
        System.out.println(username+"\n"+password+"\n"+ Arrays.toString(hobby));
        System.out.println("JSESSIONID:"+Jsessionid+"\nheader"+header);
        return "first";
    }
    }
```


@RequestHeader是将请求头信息和控制器方法的形参创建映射关系
@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestParam
@CookieValue是将cookie数据和控制器方法的形参创建映射关系
@CookieValue注解一共有三个属性：value、required、defaultValue，用法同@RequestParam
获取 Cookie 和 Header 实际上也可以用HttpServletRequest的方法获取。
## 5、通过实体类获取请求参数
可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值

```html
<form th:action="@{/first4}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    性别：<input type="radio" name="sex" value="男">男
    <input type="radio" name="sex" value="女">女
    邮箱：<input type="email" name="email"><br>
    <input type="submit" value="登录">
</form>
```
如果参数太多可以采用实体类的方式

```java
public class User {
   private String username;
   private String password;
   private String sex;
   private String email;
   。。。。。省略getter、setter、构造方法及toString等方法
   }
```

```java
    @RequestMapping(value = "first4")
    public String first4(User user){
        System.out.println(user);
        return "first";
    }
```

得到结果
User{username='11', password='22', sex='??·', email='w2@qq.com'}
会发现有 sex 的乱码，因此需要解决乱码问题
这个可以在tomcat里设置utf-8的编码方式，不过在这里将会使用过滤器的方式来解决这个问题。
通过在web.xml配置的拦截器的方式能够很好的解决这个问题。
```xml
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
# 域对象共享数据（四种）
## Request域对像
生命周期：一次请求
### 利用ServletAPIRequest

```java
    @RequestMapping(value = "/testServletRequest")
    public String testServletRequest(HttpServletRequest request){
        request.setAttribute("test","value");
        return "success";
    }
   
```
### 利用ModelAndView
```bash
    @RequestMapping(value = "/testModelAndView")
    public ModelAndView testModelAndView(){
        ModelAndView mv = new ModelAndView();
        mv.addObject("test","modelandview");
        Map<String,Integer> map = new HashMap<>();
        map.put("test1",1);
        map.put("test2",2);
        mv.addObject("test","modelandview");
        mv.addObject("map",map);
        mv.setViewName("success");
        return mv;
    }
   
```
### 利用Model
```java
   @RequestMapping("/testModel")
    public String testModel(Model model) {
        model.addAttribute("test", "hello,Model");
        return "success";
    }
```
### 利用Map
```bash
   @RequestMapping("/testMap")
    public String testMap(Map<String, Object> map) {
        map.put("map", "hello,Map");
        return "success";
    }
   
```
### 利用ModelMap
```bash
    @RequestMapping("/testModelMap")
    public String testModelMap(ModelMap modelMap){
        modelMap.addAttribute("testScope", "hello,ModelMap");
        return "success";
    }

```
分别对应的前端读取方法

```html'
ServletAPIRequest共享数据:<p th:text="${test}"></p>
ModelAndView共享数据:
<p th:text="${test}"></p>
<p th:text="${map}"></p>
Model共享数据:
<p th:text="${test}"></p>
Map共享数据:
<p th:text="${map}"></p>
ModelMap共享数据:
<p th:text="${testScope}"></p>
```
分析：
分别对上面四种 ModelView，Model，Map，ModelMap 的实现类进行分析
依次将代码放入显示

```java
System.out.println(mv.getClass().getName());
System.out.println(model.getClass().getName());
System.out.println(map.getClass().getName());
System.out.println(modelMap.getClass().getName());
```

```bash
org.springframework.web.servlet.ModelAndView
org.springframework.validation.support.BindingAwareModelMap 		
org.springframework.validation.support.BindingAwareModelMap
org.springframework.validation.support.BindingAwareModelMapp)

```
发现 Model，Map，ModelMap 是一个实现类，他们本质上使用时一回事
需要注意，Model 与 Map 是一个接口，ModelMap 继承了 LinkedHashMap；ExtendedModelMap 继承了 ModelMap ，同时实现了Model，最终 BindingAwareModelMap 实现了 ExtendedModelMap ，因此可知他们本质上是一个东西，能够互相影响。

```java
public interface Model{} 
public class ModelMap extends LinkedHashMap<String, Object> {} 
public class ExtendedModelMap extends ModelMap implements Model {}
 public class BindingAwareModelMap extends ExtendedModelMap {}
```
更有趣的时 ModelView，ModelView 是 SpringMVC 前后端能够共享数据的的核心，上面的几种方式实际上都会通过返回 ModelView 的方式来返回到前端数据，因此他们的数据实际上是共享的。
作为 request，最终实现本质上是  request.setAttribute("test","value"); 这种方式的不同表现，内部数据在一次请求是共享的。

## Session 域对象
Session 对象一次会话有效
```java
    @RequestMapping("/testSession")
    public String testSession(HttpSession session){
        session.setAttribute("test","hello,session");
        return "success";
    }
```
前端：

```html
session共享数据
<p th:text="${session.test}"></p>
```
## Application 域对象
整个项目有效
```java
    @RequestMapping("/testApplication")
    public String testApplication(HttpSession session){
        ServletContext context = session.getServletContext();
        context.setAttribute("test","hello,Context");
        return "success";
    }
```

前端:

```html
application共享数据
<p th:text="${application.test}"></p>
```
还有一个域对象仅在page页面有效，是一个作用范围最小的域对象，因为在前端页面中共享数据应用较多，本篇不再赘述。