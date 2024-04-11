
# XSS跨站点预防规则
不要简单地对各种规则中提供的示例字符列表进行编码/转义。仅编码/转义该列表是不够的。阻止列表方法非常脆弱。此处的允许列表规则经过精心设计，即使将来因浏览器更改而引入的漏洞也能提供保护。
## 规则 0  除了在允许的位置，永远不要插入不受信任的数据
第0条规则是拒绝所有不受信任的数据放入您的 HTML 文档，除非它满足下面所陈述的规则。原因是 HTML 中有太多奇怪的上下文，以至于编码规则列表变得非常复杂。例如 JavaScript 中的 URL的编码很麻烦。最重要的是，永远不要接受来自不受信任来源的实际 JavaScript 代码然后运行它。例如，名为“回调”的参数包含一个 JavaScript 代码片段，目前没有多少编码/转义可以解决这个问题。
## 规则 1 在将不可信数据插入到HTML标签之间时，对这些数据进行HTML Entity编码
规则 1 用于将不受信任的数据直接放入 HTML 正文中的某处，如下所示，在这些正文标签中需要先对插入的数据进行编码才能减少XSS攻击引起的问题。
```
<body>…插入不可信数据前，对其进行HTML Entity编码…</body>
<div>…插入不可信数据前，对其进行HTML Entity编码…</div>
<p>…插入不可信数据前，对其进行HTML Entity编码…</p>
<td>…插入不可信数据前，对其进行HTML Entity编码…<td>
以此类推，往其他HTML标签之间插入不可信数据前，对其进行HTML Entity编码
```
编码方式：
```
&     –>     &amp;
<     –>     &lt;
>     –>     &gt;
”     –>     &quot;
‘     –>     &#x27;
/     –>     &#x2f;
```
实际上，大多数 Web 框架都有一种方法来对下面详述的字符进行 HTML 编码/转义。但是，这对于其他 HTML 上下文是不够的。一般转义中没有对“/”符号进行转义，使用OWASP提供的ESAPI函数库，它提供了一系列非常严格的用于进行各种安全编码的函数。在当前这个例子里，你可以使用:
```
String encodedContent = ESAPI.encoder().encodeForHTML(untrustDate);
```

## 规则 2 在将不可信数据插入到HTML属性里时，对这些数据进行HTML属性编码
这条原则是指，当你要往HTML属性(例如width、name、value属性)的值部分(data value)插入不可信数据的时候，应该对数据进行HTML属性编码。不过需要注意的是，当要往HTML标签的事件处理属性(例如onmouseover)里插入数据的时候，本条原则不适用，应该用下面介绍的原则4对其进行JavaScript编码。
例如：
```
<div attr=…插入不可信数据前，进行HTML属性编码…></div>属性值部分没有使用引号，不推荐
<div attr=’…插入不可信数据前，进行HTML属性编码…’></div>
属性值部分使用了单引号
<div attr=”…插入不可信数据前，进行HTML属性编码…”></div>
属性值部分使用了双引号
```
解决方法：
解决方法：
```
String encodedContent = ESAPI.encoder().encodeForHTMLAttribute(request.getParameter(“input”));
```
需要注意：
1、除了阿拉伯数字和字母，对其他所有的字符进行编码，只要该字符的ASCII码小于256。编码后输出的格式为 &#xHH; (以&#x开头，HH则是指该字符对应的十六进制数字，分号作为结束符)，
2、始终用双引号“”或单引号'引用动态属性。 "和'应该分别编码为&#x22;和&#x27;。
3、带引号的属性只能用相应的引号分隔，而未带引号的属性可以用许多字符分隔，包括[space] % * + , - / ; < = > ^和|。
4、某些属性可用于带编码的攻击事件，不应是动态的，或要格外小心
5、href可用于使用javascript伪协议注入JavaScript，例如
```
href="javascript:attack()）
```
因此对于事件处理程序 ( onclick, onerror, onmouseover, ...)应该通过 JavaScript编码方式来解决（规则3）。
对于能够传递到style的属性同样需要通过CSS编码方式来解决（）。

## 规则 3 在将不可信数据插入到SCRIPT里时，对这些数据进行JavaScript编码
这条原则主要针对动态生成的JavaScript代码，这包括脚本部分以及HTML标签的事件处理属性(Event Handler，如onmouseover, onload等[（菜鸟例子）](https://www.runoob.com/jsref/dom-obj-event.html))。在往JavaScript代码里插入数据的时候，只有一种情况是安全的，那就是对不可信数据进行JavaScript编码，并且只把这些数据放到使用引号包围起来的值部分(data value)之中，例如：
```
在带引号的字符串中：
<script>alert(‘…插入不可信数据前，进行JavaScript编码…’)</script>值部分使用了单引号
<script>x = “…插入不可信数据前，进行JavaScript编码…”</script>值部分使用了双引号
在引用的事件处理程序中：
<div οnmοuseοver=”x=’…插入不可信数据前，进行JavaScript编码…’“</div>值部分使用了引号，且事件处理属性的值部分也使用了引号

```
特别需要注意的是，在XSS防御中，有些JavaScript函数是极度危险的，
就算对不可信数据进行JavaScript编码，也依然会产生XSS漏洞，例如：
```
<script>
window.setInterval(‘…就算对不可信数据进行了JavaScript编码，
这里依然会有XSS漏洞…’);</script>
```
编码规则：
除了阿拉伯数字和字母，对其他所有的字符进行编码，只要该字符的ASCII码小于256。编码后输出的格式为 \xHH (以 \x 开头，HH则是指该字符对应的十六进制数字)
不要使用任何转义快捷方式，\"因为引号字符可能会被首先运行的 HTML 属性解析器匹配。这些转义快捷方式也容易受到攻击者发送的逃逸攻击，\"易受攻击的代码将其转换\\"为启用引用的代码。
编码方式：
```
String encodedContent = ESAPI.encoder().encodeForJavaScript(untrustDate);
```
## 规则 4 在将不可信数据插入到Style属性里时，对这些数据进行CSS编码
当需要往Stylesheet，Style标签或者Style属性里插入不可信数据的时候，需要对这些数据进行CSS编码。传统印象里CSS不过是负责页面样式的，但是实际上它比我们想象的要强大许多，而且还可以用来进行各种攻击。因此，不要对CSS里存放不可信数据掉以轻心，应该只允许把不可信数据放入到CSS属性的值部分，并进行适当的编码。除此以外，最好不要把不可信数据放到一些复杂属性里，比如url, behavior等，只能被IE认识的Expression属性允许执行JavaScript脚本，因此也不推荐把不可信数据放到这里。
```
<style>
selector { property : …插入不可信数据前，进行CSS编码…}
 </style>
<style>
selector { property : ” …插入不可信数据前，进行CSS编码… “}
 </style>
<span style=” property : …插入不可信数据前，进行CSS编码… ”> … </span>
```
编码规则：
除了阿拉伯数字和字母，对其他所有的字符进行编码，只要该字符的ASCII码小于256。编码后输出的格式为 \HH (以 \ 开头，HH则是指该字符对应的十六进制数字)
同原则2，原则3，在对不可信数据进行编码的时候，切忌投机取巧对双引号等特殊字符进行简单转义，攻击者可以想办法绕开这类限制。
可以使用ESAPI提供的函数进行CSS编码：
```
String encodedContent = ESAPI.encoder().encodeForCSS(request.getParameter(“input”));
```
## 规则 5  在将不可信数据插入到HTML URL里时，对这些数据进行URL编码和验证
当需要往HTML页面中的URL里插入不可信数据的时候，需要对其进行URL编码，如下：
编码规则：
除了阿拉伯数字和字母，对其他所有的字符进行编码，只要该字符的ASCII码小于256。编码后输出的格式为 %HH (以 % 开头，HH则是指该字符对应的十六进制数字)
在对URL进行编码的时候，有两点是需要特别注意的：
1 URL属性应该使用引号将值部分包围起来，否则攻击者可以很容易突破当前属性区域，插入后续攻击代码
2 不要对整个URL进行编码，因为不可信数据可能会被插入到href, src或者其他以URL为基础的属性里，这时需要对数据的起始部分的协议字段进行验证，否则攻击者可以改变URL的协议，例如从HTTP协议改为DATA伪协议，或者javascript伪协议。

可以使用ESAPI提供的函数进行URL编码
```
<a href="http://www.somesite.com?test=...编码不受信任的数据...">link</a >
```
ESAPI还提供了一些用于检测不可信数据的函数，在这里我们可以使用其来检测不可信数据是否真的是一个URL：
```
String userURL = request.getParameter( "userURL" )
boolean isValidURL = Validator.IsValidURL(userURL, 255);
if (isValidURL) {
    <a href="<%=encoder.encodeForHTMLAttribute(userURL)%>">link</a>
}
```
## 规则 6 使用富文本时，使用XSS规则引擎进行编码过滤
Web应用一般都会提供用户输入富文本信息的功能，比如BBS发帖，写博客文章等，用户提交的富文本信息里往往包含了HTML标签，甚至是JavaScript脚本，如果不对其进行适当的编码过滤的话，则会形成XSS漏洞。但我们又不能因为害怕产生XSS漏洞，所以就不允许用户输入富文本，这样对用户体验伤害很大。
针对富文本的特殊性，我们可以使用XSS规则引擎对用户输入进行编码过滤，只允许用户输入安全的HTML标签，如`<b>, <i>, <p>`等，对其他数据进行HTML编码。需要注意的是，经过规则引擎编码过滤后的内容只能放在`<div>, <p>`等安全的HTML标签里，不要放到HTML标签的属性值里，更不要放到HTML事件处理属性里，或者放到`<SCRIPT>`标签里。
推荐XSS规则过滤引擎：OWASP AntiSamp或者Java HTML Sanitizer

## 规则 7 避免使用 JavaScript URL
包含 javascript: 协议的不受信任的 URL 在 URL DOM 位置（例如锚标记 HREF 属性或 iFrame src 位置）中使用时将执行 JavaScript 代码。请务必验证所有不受信任的 URL，以确保它们仅包含安全方案，例如 HTTPS。
##  HTML 在 HTML 上下文中编码 JSON 值并使用 JSON.parse 读取数据
在 Web 2.0 世界中，需要由应用程序在 JavaScript 进行 AJAX 调用以动态获取json数据时，通常的 JSON 块会加载到页面中。一般会进行如下操作：
```
<script>
//没有使用编码便使用
var initData = <%= data.to_json %>;
</script>
```
更好的方式是在使用前对json数据进行编码，在不破坏正常值的情况下进行转码的方法，需要注意确保返回的Content-Type标头是application/json而不是text/html。这将指示浏览器不要错误识别上下文并执行注入的脚本
可以从页面的HTTP Response去观察：
```
Content-Type: text/html; charset=utf-8 <-- bad
Content-Type: application/json; charset=utf-8 <--good
```
编码方式：
```

<div id="init_data" style="display: none">
 <%= html_encode(data.to_json) %>
</div>
```
```
// external js file
var dataElement = document.getElementById('init_data');
// decode and parse the content of the div
var initData = JSON.parse(dataElement.textContent);
```

## 其他

除了直接在JavaScript中对JSON进行编码和解码外，另一种方法是在将<转换为\u003c，然后将其传递到浏览器之前，对JSON服务器端进行规范化。
### 使用 HTTPOnly cookie 标志
### 实施内容安全策略
例如：
```
Content-Security-Policy: default-src: 'self'; script-src: 'self' static.domain.tld
```
### 使用自动转义模板系统
许多 Web 应用程序框架提供自动上下文转义功能，例如AngularJS 严格上下文转义和Go 模板。尽可能使用这些技术。
### 正确使用现代 JS 框架
现代 JavaScript 框架内置了非常好的 XSS 保护。通常框架 API 允许绕过该保护以呈现未转义的 HTML 或包含可执行代码
Angular有相关文档可以知道危险的api
### X-XSS-保护头
该X-XSS-Protection头已被弃用，现代浏览器和它的使用可以引入额外的客户端上的安全问题。因此，建议将标头设置X-XSS-Protection: 0为禁用 XSS 审计器，并且不允许它采用浏览器处理响应的默认行为。查看以下参考资料以更好地了解该主题：
Google Chrome 的 XSS Auditor 回到过滤模式
Chrome 移除了 XSS 审计器
Firefox 没有实现 XSSAuditor
Edge 停用了他们的 XSS 过滤器
OWASP ZAP 弃用了对标头的扫描
SecurityHeaders.com 不再扫描标题
# XSS 预防规则摘要
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-maDtGqrd-1646459285161)(_v_images/20211102171409801_562838233.png)]
# 输出编码规则总结
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-evROWUOu-1646459285163)(_v_images/20211102171522794_1675908460.png)]