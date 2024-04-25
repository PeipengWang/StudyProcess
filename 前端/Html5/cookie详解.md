# Cookie
## 为什么要用Cookie？
HTTP协议本身是无状态的。什么是无状态呢，即服务器无法判断用户身份。Cookie可以记录访问者的身份。
通常Cookie是记录身份的作用最简单的使用就是登录功能。
还有一种使用能够作为全局变量的实现，来跟踪用户行为。
## Cookie是什么？
客户端向服务器发起请求，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。
cookie是浏览器提供的一种机制，它将document 对象的cookie属性提供给JavaScript。可以由JavaScript对其进行控制，而并不是JavaScript本身的性质。cookie是存于用户硬盘的一个文件，这个文件通常对应于一个域名，当浏览器再次访问这个域名时，便使这个cookie可用。因此，cookie可以跨越一个域名下的多个网页，但不能跨越多个域名使用。
## Cookie怎么用？
基本流程：
当用户第一次访问并登陆一个网站的时候，cookie的设置以及发送会经历以下4个步骤：
客户端发送一个请求到服务器 --》 服务器发送一个HttpResponse响应到客户端，其中包含Set-Cookie的头部 --》 客户端保存cookie，之后向服务器发送请求时，HttpRequest请求中会包含一个Cookie的头部 --》服务器返回响应数据
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Cs9ag3EJ-1691721601938)(vx_images/303144440647148.png)]
## Cookie常用属性
一个Cookie包含以下信息：
1)Cookie名称，Cookie名称必须使用只能用在URL中的字符，一般用字母及数字，不能包含特殊字符，如有特殊字符想要转码。如js操作cookie的时候可以使用escape()对名称转码。
2)Cookie值，Cookie值同理Cookie的名称，可以进行转码和加密。
3)Expires，过期日期，一个GMT格式的时间，当过了这个日期之后，浏览器就会将这个Cookie删除掉，当不设置这个的时候，Cookie在浏览器关闭后消失。
4)Path，一个路径，在这个路径下面的页面才可以访问该Cookie，一般设为“/”，以表示同一个站点的所有页面都可以访问这个Cookie。
5)Domain，子域，指定在该子域下才可以访问Cookie，例如要让Cookie在a.test.com下可以访问，但在b.test.com下不能访问，则可将domain设置成a.test.com。
6)Secure，安全性，指定Cookie是否只能通过https协议访问，一般的Cookie使用HTTP协议既可访问，如果设置了Secure（没有值），则只有当使用https协议连接时cookie才可以被页面访问。
7)HttpOnly，如果在Cookie中设置了"HttpOnly"属性，那么通过程序(JS脚本、Applet等)将无法读取到Cookie信息。
8）samesite属性
Cookie的属性SameSite如果不配置或者配置为none，则存在CSRF风险。
SameSite的取值可以为：
unset（默认）。这种情况浏览器可能会采用自己的策略。
none。存在CSRF风险。
lax。大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。
strict。完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie
## 修改与删除：
HttpServletResponse提供的Cookie操作只有一个addCookie(Cookie cookie)，所以想要修改Cookie只能使用一个同名的Cookie来覆盖原先的Cookie。如果要删除某个Cookie，则只需要新建一个同名的Cookie，并将maxAge设置为0，并覆盖原来的Cookie即可。
新建的Cookie，除了value、maxAge之外的属性，比如name、path、domain都必须与原来的一致才能达到修改或者删除的效果。否则，浏览器将视为两个不同的Cookie不予覆盖。
值得注意的是，从客户端读取Cookie时，包括maxAge在内的其他属性都是不可读的，也不会被提交。浏览器提交Cookie时只会提交name和value属性，maxAge属性只被浏览器用来判断Cookie是否过期，而不能用服务端来判断。
 Cookie cookie = new Cookie("mcrwayfun",System.currentTimeMillis()+"");
// 设置生命周期为MAX_VALUE，永久有效、
cookie.setMaxAge(Integer.MAX_VALUE);
resp.addCookie(cookie);
## 在浏览器查看cookie
查看当前页面的cookie
f12进入application为全局cookie
当前访问网址的请求cookie与返回cookie
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/32d64b98614a442782b59ec266234aee.png)

通过命令方式也可以
f12进入console
输入：javascript:alert(document.cookie)

## 前端页面读取Cookie
写cookie

```
function writeCookie() {
document.cookie="userId=828";
//如果要一次存储多个名/值对，可以使用分号加空格（; ）隔开，例如
document.cookie="userId=828; userName=hulk";
alert("操作成功");
}
```
用escape()函数进行编码，它能将一些特殊符号使用十六进制表示，例如空格将会编码为“20%”，从而可以存储于cookie值中，而且使用此 种方案还可以避免中文乱码的出现。
例如
```
document.cookie="str="+escape("I love ajax"); 
```
当使用escape()编码后，在取出值以后需要使用unescape()进行解码才能得到原来的cookie值。
只能够一次获取所有的cookie值，而不能指定cookie名称来获得指定的值，这正是处理cookie值最麻 烦的一部分。用户必须自己分析这个字符串，来获取指定的cookie值
读cookie
```
document.cookie="userId=828";
document.cookie="userName=hulk";
//获取cookie字符串
var strCookie=document.cookie;
//将多cookie切割为多个名/值对
var arrCookie=strCookie.split("; ");
var userId;
//遍历cookie数组，处理每个cookie对
for(var i=0;i<arrCookie.length;i++){
var arr=arrCookie[i].split("=");
//找到名称为userId的cookie，并返回它的值
if("userId"==arr[0]){
userId=arr[1];
break;
}
}
alert(userId);
```
删除cookie
如果想要某个cookie失效，那么设置这个cookie的expires为过去的某个时间即可
```
//获取当前时间 
var date=new Date(); 
//将date设置为过去的时间 
date.setTime(date.getTime()-10000); 
//将userId这个cookie删除 
document.cookie="userId=828; expires="+date.toGMTString(); 
```
## jQuery Cookie 插件
引入方式

```javascript
<script src="/path/to/jquery.min.js"></script>
<script src="/path/to/jquery.cookie.js"></script>
```
### 创建
创建 cookie：

```javascript
$.cookie('name', 'value');
```

如果未指定过期时间，则会在关闭浏览器时过期。
创建 cookie，并设置 7 天后过期：
```javascript
$.cookie('name', 'value', { expires: 7 });
```

创建 cookie，并设置 cookie 的有效路径，路径为网站的根目录：
```javascript
$.cookie('name', 'value', { expires: 7, path: '/' });
```
### 读取

```javascript
$.cookie('name'); // => "value"
$.cookie('nothing'); // => undefined
```
读取所有的 cookie 信息：

```javascript
$.cookie(); // => { "name": "value" }
```

### 删除 cookie
// cookie 删除成功返回 true，否则返回 false

```javascript
$.removeCookie('name'); // => true
$.removeCookie('nothing'); // => false 
```
// 写入使用了 path时，读取也需要使用相同的属性 (path, domain) 

```javascript
$.cookie('name', 'value', { path: '/' });
```
```javascript
// 以下代码【删除失败】
$.removeCookie('name'); // => false
// 以下代码【删除成功】
$.removeCookie('name', { path: '/' }); // => true
```

## Java后端读写cookie
servlet直接设置传递：
```java
Cookie cookie = new Cookie("color", URLEncoder.encode("黄色", "UTF-8"));
Cookie name = new Cookie("name", URLEncoder.encode("zhang三", "UTF-8"));
// 设置过期时间为24小时,以秒为单位
cookie.setMaxAge(60 * 60 * 24);
name.setMaxAge(60 * 60 * 1);
// 添加cookie
response.addCookie(cookie);
response.addCookie(name);
request.getRequestDispatcher("/cookie.html").forward(request, response);
```
获取前端cookie并设置
```java
request.setCharacterEncoding("UTF-8");
Map<String, String> returnMap = new HashMap<>();
Cookie[] cookies = request.getCookies();
for (Cookie cookie : cookies) {
String name = cookie.getName();
String value = URLDecoder.decode(cookie.getValue(), "UTF-8");
returnMap.put(name, value);
}
Map<String, String[]> parameterMap = request.getParameterMap();
for (Map.Entry<String, String[]> entry : parameterMap.entrySet()) {
returnMap.put(entry.getKey(), entry.getValue()[0]);
}
```

## 最典型的cookie--JESSIONID
### 是什么？
首先，JSESSIONID是一个Cookie，Servlet容器（tomcat，jetty）用来记录用户session。
一般cookie中都存在一个sessionID，每种语言使用存储sessionID的默认cookie名称都不太一样，具体如下：
asp：ASPSESSIONID
php：PHPSESSIONID
jsp：JSESSIONID
### 什么时候种下JSESSIONID？
创建会话时，即调用request.getSession()的时候，关于getSession就不说了。访问html是不会创建session的，JSP页面默认是会创建session的，可以在JSP页面里面关掉自动创建session
### 怎么用？
在注销登录，登录超时添加：
```
//HttpServletRequest request
HttpSession session = request.getSession();
session.invalidate();
```
 session.invalidate()是将session设置为失效，一般在退出时使用，但要注意的是：session失效的同时 浏览器会立即创建一个新的session  ,JSESSIONID 只是tomcat中对session id的叫法，在其它容器里面，不一定就是叫JSESSIONID 了
session.invalidate()执行此代码时 JSESSIONID 会失效，浏览器会立即创建一个新的JSESSIONID。
调用Request对象的isRequestedSessionIdFromCookie判断客户端Cookie是否可用，里面逻辑也很简单，就是读取request里面有没有传JSESSIONID这个Cookie。所以网上有些人说第一次访问，其实只要客户端没有传JSESSIONID，tomcat都假定Cookie不可用。
```
jsessionid = request.getSession().getId()
sessio（获取session并将数据更新
// 开始一个新的 session
         HttpSession session = req.getSession();
         //首先将原session中的数据转移至一临时map中
         Map<String,Object> tempMap = new HashMap();
         Enumeration<String> sessionNames = session.getAttributeNames();
         while(sessionNames.hasMoreElements()){
             String sessionName = sessionNames.nextElement();
             tempMap.put(sessionName, session.getAttribute(sessionName));
             System.out.println(sessionName);
         }
         //注销原session，为的是重置sessionId
         session.invalidate();
         //将临时map中的数据转移至新session
         session = req.getSession();
         for(Map.Entry<String, Object> entry : tempMap.entrySet()){
             session.setAttribute(entry.getKey(), entry.getValue());
```
## Cookie的使用限制
WEB开发时cookie大小是受到限制的
1、浏览器允许每个域名所包含的cookie数：
　　Microsoft指出InternetExplorer8增加cookie限制为每个域名50个，但IE7似乎也允许每个域名50个cookie。
　　Firefox每个域名cookie限制为50个。
　　Opera每个域名cookie限制为30个。
　　Safari/WebKit貌似没有cookie限制。但是如果cookie很多，则会使header大小超过服务器的处理的限制，会导致错误发生。
　　注：“每个域名cookie限制为20个”将不再正确！但是为了兼容低版本，我们尽量将cookie数控制在20个或20个以下。
2、当很多的cookie被设置，浏览器如何去响应。
　　除Safari（可以设置全部cookie，不管数量多少），有两个方法：
　　最少最近使用（leastrecentlyused(LRU)）的方法：当Cookie已达到限额，自动踢除最老的Cookie，以使给最新的Cookie一些空间。InternetExplorer和Opera使用此方法。
　　Firefox很独特：虽然最后的设置的Cookie始终保留，但似乎随机决定哪些cookie被保留没有任何计划。
　　建议：在Firefox中不要超过Cookie限制
3、不同浏览器间cookie总大小也不同：
　　Firefox和Safari允许cookie多达4097个字节，包括名（name）、值（value）和等号。
　　Opera允许cookie多达4096个字节，包括：名（name）、值（value）和等号。
　　InternetExplorer允许cookie多达4095个字节，包括：名（name）、值（value）和等号。
　　注意：多字节字符计算为两个字节。在所有浏览器中，任何cookie大小超过限制都被忽略，且永远不会被设置。这样的话，为了兼容低版本，我们尽量控制cookie的大小为4095或4095以下
## Cookie的安全性策略
1、对保存到cookie里面的敏感信息必须加密
2、设置HttpOnly为true
　　1）、该属性值的作用就是防止Cookie值被页面脚本读取，换句话说，当cookie的HTTPOnly属性被设为True时（默认为false），document对象中就找不到该cookie了。
　　2）、但是设置HttpOnly属性，HttpOnly属性只是增加了攻击者的难度，Cookie盗窃的威胁并没有彻底消除，因为cookie还是有可能传递的过程中被监听捕获后信息泄漏。
      3）、网站中存储sessionID的cookie一定要设置。这也是AWVS为何命名该漏洞为Session Cookie without Secure flag set的原因。一般网站应用也不会在js里操作这些敏感Cookie的，对于一些需要在应用程序中用JS操作的cookie我们就不予设置。这样就保障了Cookie信息的安全，同时的保证了网站的正常业务。
2、设置Secure为true
　　1）、给Cookie设置该属性时，只有在https协议下访问的时候，浏览器才会发送该Cookie。
      2）、把cookie设置为secure，只保证cookie与WEB服务器之间的数据传输过程加密，而保存在本地的cookie文件并不加密。如果想让本地cookie也加密，得自己加密数据。
3、给Cookie设置有效期
　　 1）、如果不设置有效期，万一用户获取到用户的Cookie后，就可以一直使用用户身份登录。
    　2）、在设置Cookie认证的时候，需要加入两个时间，一个是“即使一直在活动，也要失效”的时间，一个是“长时间不活动的失效时间”，并在Web应用中，首先判断两个时间是否已超时，再执行其他操作。
4、一定不要在cookie中存储重要和敏感的数据，存储要进行加密。
## 过多的 Cookie 会带来巨大的性能浪费
Cookie 是紧跟域名的。同一个域名下的所有请求，都会携带 Cookie。大家试想，如果我们此刻仅仅是请求一张图片或者一个 CSS 文件，我们也要携带一个 Cookie 跑来跑去（关键是 Cookie 里存储的信息并不需要），这是一件多么劳民伤财的事情。Cookie 虽然小，请求却可以有很多，随着请求的叠加，这样的不必要的 Cookie 带来的开销将是无法想象的。
cookie是用来维护用户信息的，而域名(domain)下所有请求都会携带cookie，但对于静态文件的请求，携带cookie信息根本没有用，此时可以通过cdn（存储静态文件的）的域名和主站的域名分开来解决。 - 由于在HTTP请求中的Cookie是明文传递的，所以安全性成问题，除非用HTTPS。

## 存储方式
比较古老的存储方式：
    cookie
    userDate(ie5.0)
html5 存储三种方式
    local storage (永久保存，保存在缓存里。清除方法：手动清除或者浏览器缓存失效)
    application cache
    indexed DB





