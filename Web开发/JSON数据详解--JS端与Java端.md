# JSON
JSON实际上是JavaScript的一个子集。在JSON中，一共就这么几种数据类型：

number：和JavaScript的number完全一致；  
boolean：就是JavaScript的true或false；  
string：就是JavaScript的string；  
null：就是JavaScript的null；  
array：就是JavaScript的Array表示方式——[]；  
object：就是JavaScript的{ ... }表示方式。  
JSON还定死了字符集必须是UTF-8，表示多语言就没有问题了  
把任何JavaScript对象变成JSON，就是把这个对象序列化成一个JSON格式的字符串，这样才能够通过网络传递给其他计算机。   
如果我们收到一个JSON格式的字符串，只需要把它反序列化成一个JavaScript对象，就可以在JavaScript中直接使用这个对象了。  

## 语法
数据在名称/值对中  
数据由逗号分隔  
大括号 {} 保存对象  
中括号 [] 保存数组，数组可以包含多个对象  
其中对象包括键值对（属性:属性值）{key： value}，value 可为 str，num，list，obj。取值使用 objcet.key  
{key: value, key2:value2，} 键：值用冒号分开，对间用，连接数组包含元素：num，str，list，objcet 都可以，利用索引访问 [index]，用 . 连接各个值  
### JSON 数据的书写格式  
key : value  
名称/值对包括字段名称（在双引号中），后面写一个冒号，然后是值：  
"name" : "菜鸟教程"  
这很容易理解，等价于这条 JavaScript 语句：  
name = "菜鸟教程"  
### JSON 对象中可以包含另外一个 JSON 对象：
```
myObj = {
    "name":"runoob",
    "alexa":10000,
    "sites": {
        "site1":"www.runoob.com",
        "site2":"m.runoob.com",
        "site3":"c.runoob.com"
    }
}
```
### JSON 文件
JSON 文件的文件类型是 .json  
JSON 文本的 MIME 类型是 application/json  
## 操作对象值  
### 你可以使用点号（.）来访问对象的值  
```
var myObj, x;
myObj = { "name":"runoob", "alexa":10000, "site":null };
x = myObj.name;
```
### 也可以使用中括号（[]）来访问对象的值：  
```
var myObj, x;
myObj = { "name":"runoob", "alexa":10000, "site":null };
x = myObj["name"];
```
### 循环对象  
你可以使用 for-in 来循环对象的属性：  
```
var myObj = { "name":"runoob", "alexa":10000, "site":null };
for (x in myObj) {
    document.getElementById("demo").innerHTML += x + "<br>";
}
```
修改对象同样也可以通过上述两种方式直接修改  
### 删除对象属性  
我们可以使用 delete 关键字来删除 JSON 对象的属性：  
```
delete myObj.sites.site1;
```
你可以使用中括号([])来删除 JSON 对象的属性：  
```
delete myObj.sites["site1"]
```
delete 运算符并不是彻底删除元素，而是删除它的值，但仍会保留空间。  
运算符 delete 只是将该值置为 undefined，而不会影响数组长度，即将其变为稀疏数组  
## 数组
```
{
"name":"网站",
"num":3,
"sites":[ "Google", "Runoob", "Taobao" ]
}
```
我们可以使用索引值来访问数组：  
x = myObj.sites[0];  
你可以使用 for-in 来访问数组：  
```
for (i in myObj.sites) {
    x += myObj.sites[i] + "<br>";
}
```
你也可以使用 for 循环：  
```
for (i = 0; i < myObj.sites.length; i++) {
    x += myObj.sites[i] + "<br>";
}
```
## JSON.parse()  
JSON 通常用于与服务端交换数据。  
在接收服务器数据时一般是字符串。  
我们可以使用 JSON.parse() 方法将数据转换为 JavaScript 对象。  
### 语法
```
JSON.parse(text[, reviver])
```
text:必需， 一个有效的 JSON 字符串。  
reviver: 可选，一个转换结果的函数， 将为对象的每个成员调用此函数。  
### 注意
JSON 不能存储 Date 对象。如果你需要存储 Date 对象，需要将其转换为字符串。之后再将字符串转换为 Date 对象。  
JSON 不允许包含函数，但你可以将函数作为字符串存储，之后再将字符串转换为函数。  
## JSON.stringify()  
JSON 通常用于与服务端交换数据。  
在向服务器发送数据时一般是字符串。  
我们可以使用 JSON.stringify() 方法将 JavaScript 对象转换为字符串。  
value:  
必需， 要转换的 JavaScript 值（通常为对象或数组）。  
replacer:  
可选。用于转换结果的函数或数组。  
如果 replacer 为函数，则 JSON.stringify 将调用该函数，并传入每个成员的键和值。使用返回值而不是原始值。如果此函数返回 undefined，则排除成员。根对象的键是一个空字符串：""。  
如果 replacer 是一个数组，则仅转换该数组中具有键值的成员。成员的转换顺序与键在数组中的顺序一样。当 value 参数也为数组时，将忽略 replacer 数组。      
space:    
可选，文本添加缩进、空格和换行符，如果 space 是一个数字，则返回值文本在每个级别缩进指定数目的空格，如果 space 大于 10，则文本缩进 10 个空格。space 也可以使用非数字，如：\t。    
## 转换文本对象     
JSON 语法是 JavaScript 语法的子集，JavaScript 函数 eval() 可用于将 JSON 文本转换为 JavaScript 对象。  
eval() 函数使用的是 JavaScript 编译器，可解析 JSON 文本，然后生成 JavaScript 对象。必须把文本包围在括号中，这样才能避免语法错误：  
var obj = eval ("(" + txt + ")");  
eval() 函数可编译并执行任何 JavaScript 代码。这隐藏了一个潜在的安全问题，这是必须要注意的。  
使用 JSON 解析器将 JSON 转换为 JavaScript 对象是更安全的做法。JSON 解析器只能识别 JSON 文本，而不会编译脚本。    
在浏览器中，这提供了原生的 JSON 支持，而且 JSON 解析器的速度更快。  
较新的浏览器和最新的 ECMAScript (JavaScript) 标准中均包含了原生的对 JSON 的支持。  
# Java中使用Json  
## 类库选择  
Java中并没有内置JSON的解析，因此使用JSON需要借助第三方类库。  
下面是几个常用的 JSON 解析类库：  
Gson: 谷歌开发的 JSON 库，功能十分全面。  
FastJson: 阿里巴巴开发的 JSON 库，性能十分优秀。  
Jackson: 社区十分活跃且更新速度很快。  
## FastJson的基本使用
JSON.parseObject()：从字符串解析 JSON 对象  
JSON.parseArray()：从字符串解析 JSON 数组  
JSON.toJSONString(obj/array)：将 JSON 对象或 JSON 数组转化为字符串  
String转对象  
```
Student stu = JSON.parseObject(json,Student.class);  
List<String> list=JSON.parseArray(json2, String.class);  
```
对象转String  
```
JSON.toJSONString(stu);  
//or String json = JSON.toJSON(stu).toString();  
```
