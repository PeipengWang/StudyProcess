
# 转义（escape）、校验与编码（encode）
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/017460a54baf4a8dabf36c6c8fcbb75f.png)

原则： 对输入进行转义，对输出进行校验和编码
基于DOM的跨站点脚本
JS通过DOM方式输出到HTML元素相关内容时，需要做HTMLEncode
JS通过DOM方式输出到JS元素相关内容时，需要做JavaScriptEncode
JS对所有输入进行转义和检查
使用.innerHtml时，如果只是要显示文本内容，必须在innerHtml取得内容后，去除HTML标签
客户端对页面的任何用户输入进行HTML转义和校验
服务端对客户端传入的用户数据进行进行HTML转义
## 转义（escape）
对输入直接转义会不会导致放入数据库的数据变为转义后的码
[转义工具](http://tools.jb51.net/transcoding/html_transcode)
```
<         &lt;
>         &gt;
"         &quot;
'         &#x27;
&         &amp;
/         &#2F
```
### 一种方案
```
function htmlEncode(text) {
    var encode = $('<div/>').text(text).html();
    return encode;
}
```
### 直接进行转义
```
function htmlEncode(str){
    var temp = "";
    if(str.length == 0) return "";
    temp = str.replace(/&/g,"&amp;");
    temp = temp.replace(/</g,"&lt;");
    temp = temp.replace(/>/g,"&gt;");
    temp = temp.replace(/\s/g,"&nbsp;");
    temp = temp.replace(/\'/g,"&#39;");
    temp = temp.replace(/\"/g,"&quot;");
    temp = temp.replace(/\//g,"&#x2f;");
    return temp;
}

```
### 转义库进行转义
JS对所有的输入利用转义库进行转义
JQuery Html Encoding、Decoding
 原理是利用JQuery自带的html()和text()函数可以转义Html字符
虚拟一个Div通过赋值和取值来得到想要的Html编码或者解码
#### JQuery Html Encoding、Decoding
原理是利用JQuery自带的html()和text()函数可以转义Html字符
虚拟一个Div通过赋值和取值来得到想要的Html编码或者解码
```
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script type="text/javascript">
//Html编码获取Html转义实体
function htmlEncode(value){
  return $('<div/>').text(value).html();
}
//Html解码获取Html实体
function htmlDecode(value){
  return $('<div/>').html(value).text();
}
```
实例：idea
#### StringEscapeUtils
Java中用于转义处理的类有Apache commons-lang 包中的StringEscapeUtils、spring-web 包中的HtmlUtils、JavaScriptUtils等。
来源包：org.apache.commons
项目中已有这个包

```
import org.apache.commons.lang.StringEscapeUtils;

public class decode {

    public static void main(String args[]){
        String input = StringEscapeUtils.escapeHtml("左三角<右三角>test 双引号”单引号'反斜杠/</font>");
        System.out.println("转义HTML,注意汉字:"+input); 	//转义HTML,注意汉字
        System.out.println("反转义HTML:"+StringEscapeUtils.unescapeHtml(input));	//反转义HTML
    }
}
```
会将中文字符&#形式
前后端都可以直接用
```
&#24038;&#19977;&#35282;&lt;&#21491;&#19977;&#35282;&gt;test &#21452;&#24341;&#21495;&rdquo;&#21333;&#24341;&#21495;'&#21453;&#26012;&#26464;/&lt;/font&gt;
```
## 校验
校正：避免直接对HTML Entity编码，使用DOM Prase转换，校正不配对的DOM标签。

### 服务端校验
```
方法入参如下：
boolean isXSSValid(String str, int maxLength, boolean allowNull)
使用示例
boolean isValid = isXSSValid("<img src='s' onerror=alert(1)<!--'",1000,false);
```
## 编码（encode）
使用“\”对特殊字符进行转义，除数字字母之外，小于127的字符编码使用16进制“\xHH”的方式进行编码，大于用unicode（非常严格模式）。
关于如何编码，需要抓住一个重点，就是传进的值最终是谁消费（期望执行者），谁消费谁负责；对于支持伪协议的属性，仅仅通过编码是不行的，需要匹配过滤协议头，最好后台拆分拼接返回期望的值。（不能对协议类型进行任何的编码操作，否则URL解析器会认为它无类型）。
造成DOM XSS的原因主要是在重新修改页面时，没有考虑到对变量的编码和校验过滤；
该OWASP的Java编码器项目为Java提供了高性能的编码库。

### 需要编码的位置
#### HTML 实体编码
不可信数据插入实体中
```
1. & —> &amp;
2. < —> &lt;
3. > —> &gt;
4. " —> &quot;
5. ' —> &#x27;
6. / —> &#x2F;
```
```
String encodedContent = 
ESAPI.encoder().encodeForHTML(request.getParameter(“input”));
```
#### HTML 属性编码
#### 网址编码
 标准百分比编码，URL 编码应仅用于对参数值进行编码，而不是对整个 URL 或 URL 的路径片段进行编码。
#### JavaScript 编码
   除字母数字字符外，使用\uXXXXunicode 编码格式 ( X = Integer)对所有字符进行编码。
   使用“\”对特殊字符进行转义，除数字字母之外，小于127的字符编码使用16进制“\xHH”的方式进行编码，大于用unicode（非常严格模式）。
#### CSS 十六进制编码
   CSS 编码支持\XX和\XXXXXX. 如果下一个字符继续编码序列，则使用两个字符编码可能会导致问题。有两种解决方案 (a) 在 CSS 编码后添加一个空格（将被 CSS 解析器忽略）（b）通过零填充值来使用可能的全部 CSS 编码。
### 编码方案
#### JavaScript 编码库
js对文字进行编码涉及3个函数：escape,encodeURI,encodeURIComponent，相应3个解码函数：unescape,decodeURI,decodeURIComponent
escape()除了 ASCII 字母、数字和特定的符号外，对传进来的字符串全部进行转义编码，因此如果想对URL编码，最好不要使用此方法。
encodeURI() 用于编码整个URI,因为URI中的合法字符都不会被编码转换。
encodeURIComponent方法在编码单个URIComponent（指请求参数）应当是最常用的，它可以讲参数中的中文、特殊字符进行转义，而不会影响整个URL。
 
#### ESAPI库编码
在构建模板化 JavaScript 时，在进入应用程序时始终 JavaScript 将不受信任的数据编码和分隔为带引号的字符串
```
var x = "<%= Encode.forJavaScript(untrustedData) %>";
```
```
ESAPI.encoder().encodeForHTML()
```
JS编码
```
encoder().encodeForJS()
```
实例
```
 var ESAPI = require('node-esapi');
 element.innerHTML = "<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>";
 element.outerHTML = "<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>";
 document.write("<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>");
 document.writeln("<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>");
```


## innerHtml，而是使用innerText或textContent
如果想使用用户输入来写入div tag元素，请不要使用innerHtml，而是使用innerText或textContent。这样就解决了问题，也是修复基于DOM的XSS漏洞的正确方法。
```
<script>
document.getElementById("contentholder").textContent = document.baseURI;
</script>
```

(sum(cpuratio))/(0.0001 + sum(memratio))>'0
