# javaScript基础知识

## $的作用
如果在jquery框架里面的话它代表jquery本身。
其它时候它只是一个变量名，仅此而已。
比如
var $ = function(id)
{
return document.getElementById(id);
};
那么现在$就代表一个函数了，直接$('myDiv');就等同于document.getElementById('myDiv');
## val()
val() 方法返回或设置被选元素的 value 属性。
当用于返回值时：
该方法返回第一个匹配元素的 value 属性的值。
当用于设置值时：
该方法设置所有匹配元素的 value 属性的值。
注意：val() 方法通常与 HTML 表单元素一起使用。
```
$('#txt').val(message);
```
## attr()
attr() 方法设置或返回被选元素的属性和值。
当该方法用于返回属性值，则返回第一个匹配元素的值。
当该方法用于设置属性值，则为匹配元素设置一个或多个属性/值对。
```
$("#text_id").attr("value",message);
```
### 语法
返回属性的值：
$(selector).attr(attribute)
设置属性和值：
$(selector).attr(attribute,value)
使用函数设置属性和值：
$(selector).attr(attribute,function(index,currentvalue))
设置多个属性和值：
$(selector).attr({attribute:value, attribute:value,...})
## html()
### 定义和用法
html() 方法设置或返回被选元素的内容（innerHTML）。
当该方法用于返回内容时，则返回第一个匹配元素的内容。
当该方法用于设置内容时，则重写所有匹配元素的内容。
提示：如只需设置或返回被选元素的文本内容，请使用 text() 方法。
### 语法
返回内容：
$(selector).html()
设置内容：
$(selector).html(content)
使用函数设置内容：
$(selector).html(function(index,currentcontent))
```
$("#id").html(message);
```
## getElementById

### 定义和用法
getElementById() 方法可返回对拥有指定 ID 的第一个对象的引用。
HTML DOM 定义了多种查找元素的方法，除了 getElementById() 之外，还有 getElementsByName() 和 getElementsByTagName()。
如果没有指定 ID 的元素返回 null
如果存在多个指定 ID 的元素则返回第一个。
如果需要查找到那些没有 ID 的元素，你可以考虑通过CSS选择器使用 querySelector()。
直接赋值之前需要对message进行阻断
```
document.getElementById("span").value=message；
document.getElementById("span").innerText=message;
document.getElementById("span").innerHTML=message；
```
### value()属性
定义和用法
value 属性可设置或者返回文本域的 value 属性值。
value 属性包含了默认值或用户输入的值（或通过脚本设置）。
### innerHTML定义和用法
innerHTML 属性设置或返回表格行的开始和结束标签之间的 HTML。
语法
HTMLElementObject.innerHTML=text
## append() 
### 定义和用法
append() 方法在被选元素的结尾插入指定内容。
提示：如需在被选元素的开头插入内容，请使用 prepend() 方法。
### 语法
$(selector).append(content,function(index,html))
参数	描述
content	必需。规定要插入的内容（可包含 HTML 标签）。
可能的值：
HTML 元素
jQuery 对象
DOM 元素
function(index,html)	可选。规定返回待插入内容的函数。
index - 返回集合中元素的 index 位置。
html - 返回被选元素的当前 HTML。
## window.onload 
与 jQuery ready() 区别
window.onload = function () {};    // JavaScript 
$(document).ready(function () {}); // jQuery
以上两种方式都是在 HTML 文档完毕后再执行 DOM 操作，但它们还是有一定的区别，如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b0a682f9246e4e558ae06ae89705f4ab.png)
## 字段处理
### repalce
replace() 方法返回一个由替换值（replacement）替换部分或所有的模式（pattern）匹配项后的新字符串。模式可以是一个字符串或者一个正则表达式，替换值可以是一个字符串或者一个每次匹配都要调用的回调函数。如果pattern是字符串，则仅替换第一个匹配项。
语法：
str.replace(regexp|substr, newSubStr|function)
regexp (pattern)
一个RegExp 对象或者其字面量。该正则所匹配的内容会被第二个参数的返回值替换掉。
substr (pattern)
一个将被 newSubStr 替换的 字符串。其被视为一整个字符串，而不是一个正则表达式。仅第一个匹配项会被替换。
newSubStr (replacement)
用于替换掉第一个参数在原字符串中的匹配部分的字符串。该字符串中可以内插一些特殊的变量名。参考下面的使用字符串作为参数。
一个部分或全部匹配由替代模式所取代的新的字符串。
测试语句
```
const p = 'The quick';
console.log(p.replace(' ', ''));
// expected output: "The quick brown fox jumps over the lazy monkey. If the dog reacted, was it really lazy?"
const regex = /Dog/i;
console.log(p.replace(/(^\s*)|(\s*$)/g, ""));
```



