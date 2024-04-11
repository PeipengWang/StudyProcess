
# 防止基于 DOM 的 XSS
## 规则1
 原则：HTML 转义，然后 JavaScript 转义，然后再将不受信任的数据插入到执行上下文中的 HTML 子上下文中
 有多种方法和属性可用于在 JavaScript 中直接呈现 HTML 内容。这些方法构成了执行上下文中的 HTML 子上下文。如果这些方法提供了不受信任的输入，则可能会导致 XSS 漏洞
属性 
```
 element.innerHTML = "<HTML> Tags and markup";
 element.outerHTML = "<HTML> Tags and markup";
```
方法
```
 document.write("<HTML> Tags and markup");
 document.writeln("<HTML> Tags and markup");
```
方案：
1、HTML 编码
2、JavaScript 编码
所有不受信任的输入
```
 var ESAPI = require('node-esapi');
 element.innerHTML = "<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>";
 element.outerHTML = "<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>";
```
```
 var ESAPI = require('node-esapi');
 document.write("<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>");
 document.writeln("<%=ESAPI.encoder().encodeForJS(ESAPI.encoder().encodeForHTML(untrustedData))%>");
```
## 规则 2
在执行上下文中将不受信任的数据插入 HTML 属性子上下文之前进行 JavaScript 转义
的HTML属性的子上下文的内执行上下文是从标准编码规则发散。这是因为在 HTML 属性呈现上下文中对 HTML 属性编码的规则是必要的，以减轻尝试退出 HTML 属性或尝试添加可能导致 XSS 的其他属性的攻击。
## 规则 3 
在执行上下文中将不受信任的数据插入事件处理程序和 JavaScript 代码子上下文时要小心
将动态数据放入 JavaScript 代码尤其危险，因为与其他编码相比，JavaScript 编码对 JavaScript 编码的数据具有不同的语义。在许多情况下，JavaScript 编码不会阻止执行上下文中的攻击。
例如，即使是 JavaScript 编码的字符串，也会执行 JavaScript 编码的字符串。
因此，主要建议是避免在此上下文中包含不受信任的数据。如果必须，以下示例描述了一些有效和无效的方法。
例子
```
var x = document.createElement("a");
x.href="#";
// In the line of code below, the encoded data on the right (the second argument to setAttribute)
// is an example of untrusted data that was properly JavaScript encoded but still executes.
x.setAttribute("onclick", "\u0061\u006c\u0065\u0072\u0074\u0028\u0032\u0032\u0029");
var y = document.createTextNode("Click To Test");
x.appendChild(y);
document.body.appendChild(x);
```
该setAttribute(name_string,value_string)方法很危险，因为它隐式地将value_string 强制转换为name_string的 DOM 属性数据类型。

## 规则 4 
在执行上下文中将不受信任的数据插入 CSS 属性子上下文之前进行 JavaScript 转义
## 规则 5 
在将不受信任的数据插入执行上下文中的 URL 属性子上下文之前，先进行 URL 转义，然后进行 JavaScript 转义

## 规则 6 
使用安全的 JavaScript 函数或属性填充 DOM
使用不受信任的数据填充 DOM 的最基本的安全方法是使用安全分配属性textContent。
例如：这是一个安全的函数
```
<script>
element.textContent = untrustedData;  //does not execute code
</script>
```
## 规则 7 
修复 DOM 跨站脚本漏洞
修复基于 DOM 的跨站点脚本的最佳方法是使用正确的输出方法（接收器）。例如，如果想使用用户输入来写入div tag元素，请不要使用innerHtml，而是使用innerText或textContent。这样就解决了问题，也是修复基于DOM的XSS漏洞的正确方法。
```
<b>Current URL:</b> <span id="contentholder"></span>
...
<script>
document.getElementById("contentholder").textContent = document.baseURI;
</script>

```
在 eval 等危险源中使用用户控制的输入总是一个坏主意。在 99% 的情况下，这表明编程习惯不好或懒惰，所以不要这样做，而不要尝试清理输入。
# 使用 JavaScript 开发安全应用程序的指南
## 指南 1  
不可信数据应仅被视为可显示文本，避免将不受信任的数据视为 JavaScript 代码中的代码或标记。
## 指南 2 
在构建模板化 JavaScript 时，在进入应用程序时始终 JavaScript 将不受信任的数据编码和分隔为带引号的字符串
## 指南 3 
使用 document.createElement("...")、element.setAttribute("...","value")、element.appendChild(...) 和类似的方法来构建动态接口。
document.createElement("...")，element.setAttribute("...","value")，element.appendChild(...)和类似的是安全的办法，建立动态界面。
注意，element.setAttribute仅对有限数量的属性是安全的。
危险属性包括作为命令执行上下文的任何属性，例如onclick或onblur。
安全属性的例子包括：align，alink，alt，bgcolor，border，cellpadding，cellspacing，class，color，cols，colspan，coords，dir，face，height，hspace，ismap，lang，marginheight，marginwidth，multiple，nohref，noresize，noshade，nowrap，ref，rel，rev，rows，rowspan，scrolling，shape，span，summary，tabindex，title，usemap，valign，value，vlink，vspace，width。
## 指南 4 
避免将不受信任的数据发送到 HTML 渲染方法中
避免使用不受信任的数据填充以下方法。

```
element.innerHTML = "...";
element.outerHTML = "...";
document.write(...);
document.writeln(...);
```

## 指南 5 
避免使用大量隐式传递给它的 eval() 数据的方法
有许多方法eval()必须避免隐式传递给它的数据。
确保传递给这些方法的任何不受信任的数据是：
1、用字符串分隔符分隔
2、包含在闭包内或根据使用情况编码为 N 级的 JavaScript
3、封装在自定义函数中。
按照 3 操作确保不受信任的数据不会发送到自定义函数中的危险方法或通过添加额外的编码层来处理它。
Utilizing an Enclosure (as suggested by Gaz)
```
 var ESAPI = require('node-esapi');
 setTimeout((function(param) { return function() {
          customFunction(param);
        }
 })("<%=ESAPI.encoder().encodeForJS(untrustedData)%>"), y);
```
N 级编码
如果您的代码如下所示，您只需要对输入数据进行双重 JavaScript 编码。
```
setTimeout("customFunction('<%=doubleJavaScriptEncodedData%>', y)");
function customFunction (firstName, lastName)
     alert("Hello" + firstName + " " + lastNam);
}
```
在doubleJavaScriptEncodedData具有的JavaScript编码的它的第一层中的单引号反转（在执行时）。
然后隐式eval的setTimeout反转了另一层 JavaScript 编码以将正确的值传递给customFunction
您只需要双倍 JavaScript 编码的原因是该customFunction函数本身并没有将输入传递给另一个隐式或显式调用的方法eval如果firstName被传递给另一个隐式或显式调用的 JavaScript 方法，eval()则<%=doubleJavaScriptEncodedData%>上面需要更改为<%=tripleJavaScriptEncodedData%>.
一个重要的实现说明是，如果 JavaScript 代码尝试在字符串比较中使用双重或三重编码数据，则根据evals()传递给 if 比较和值被 JavaScript 编码的次数。
客户端编码（使用 JavaScript 编码库，如node-esapi）为不可信数据传递到的单个子上下文。
以下是它们如何使用的一些示例：
```
//server-side encoding
var ESAPI = require('node-esapi');
var input = "<%=ESAPI.encoder().encodeForJS(untrustedData)%>";
```
```
//HTML encoding is happening in JavaScript
var ESAPI = require('node-esapi');
document.writeln(ESAPI.encoder().encodeForHTML(input));
```
## 指南 6 
仅在表达式的右侧使用不受信任的数据
仅在表达式的右侧使用不受信任的数据，尤其是看起来像代码并且可能传递给应用程序的数据（例如，location和eval()）。
```window[userDataOnLeftSide] = "userDataOnRightSide";
```
## 指南 7 
在 DOM 中进行 URL 编码时要注意字符集问题。
当 DOM 中的 URL 编码时，请注意字符集问题，因为 JavaScript DOM 中的字符集没有明确定义（Mike Samuel）。

## 指南 8 
使用 object[x] 访问器时限制对对象属性的访问
使用访问器时限制对对象属性的object[x]访问（Mike Samuel）。换句话说，在不受信任的输入和指定的对象属性之间添加一个间接级别。
实例
```
var myMapType = {};
myMapType[<%=untrustedData%>] = "moreUntrustedData";
```
编写上述代码的开发人员试图向myMapType对象添加额外的键控元素。但是，攻击者可以使用它来破坏myMapType对象的内部和外部属性。
更好的方法：
```
if (untrustedData === 'location') {
  myMapType.location = "moreUntrustedData";
}
```
## 指南 10 
不要使用 eval() JSON 将其转换为原生 JavaScript 对象
不要使用eval()JSON 将其转换为原生 JavaScript 对象。而是使用JSON.toJSON()和JSON.parse()（克里斯施密特）

## 编码误解
许多安全培训课程和论文提倡盲目使用 HTML 编码来解决 XSS。这在逻辑上似乎是谨慎的建议，因为 JavaScript 解析器不理解 HTML 编码。但是，如果从您的 Web 应用程序返回的页面使用 的内容类型text/xhtml或文件类型扩展名，*.xhtml则 HTML 编码可能无法缓解 XSS。

实例
```
<form name="myForm" ...>
  <input type="text" name="lName" value="<%=ESAPI.encoder().encodeForHTML(last_name)%>">
 ...
</form>
<script>
  var x = document.myForm.lName.value;  //when the value is retrieved the encoding is reversed
  document.writeln(x);  //any code passed into lName is now executable.
</script>
```

最容易受攻击的代码
```
<script>
var x = location.hash.split("#")[1];
document.write(x);
</script>
```