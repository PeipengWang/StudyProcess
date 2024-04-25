## jQuery
### 使用
jquery.com下载jquery文件
直接从网站载入jquery库

```html
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"> </script>
```
是一个javaScript的库，里面存在大量的库函数
下载库并放到lib目录中，跟引用平常的js文件方式相同。
### 公式
$(选择器).action()
选择器为css的选择器，action是要执行的动作
选择器
例如以下三个选择器：
标签选择器（tag）
$('p').click()
id选择器（id）
$('#id').click()
类选择器（class）
$('.class').click()
### 事件
#### 文件就绪事件

```javascript
$(document).ready(function(){ // 开始写 jQuery 代码... });
```

简写：

```javascript
$(function(){ // 开始写 jQuery 代码... });
```

#### 单击事件

```javascript
$("#button1").click(function(){
    事件
 });
```

#### 双击事件

```javascript
$("p").dblclick(function(){ 事件 });
```



鼠标事件：按下，移动，离开，鼠标经过
效果
隐藏：hide()
显示：show()
语法：

```javascript
$(selector).hide(speed,callback);
$(selector).show(speed,callback);
```

可选的 speed 参数规定隐藏/显示的速度，可以取以下值："slow"、"fast" 或毫秒。
可选的 callback 参数是隐藏或显示完成后所执行的函数名称。
淡入淡出

```javascript
fadeIn()
fadeOut()
adeToggle()
fadeTo()
```

#### 滑动

```javascript
$(selector).slideDown(speed,callback);
```

向下滑动元素

```javascript
$(selector).slideUp(speed,callback);
```

向上划定元素

```javascript
$(selector).slideToggle(speed,callback);
```

切换上述两种方法
可选的 speed 参数规定效果的时长。它可以取以下值："slow"、"fast" 或毫秒。
可选的 callback 参数是滑动完成后所执行的函数名称。
## 操作Dom元素
### 捕获与设置文本

```javascript
$(selector).text()获得值
$(selector).text(”设置值”)设置值
$(selector).html()获得值
$(selector).html(“<h1>标题”)设置值
$(selector).val() 设置或返回表单字段的值
```

### 添加元素

```javascript
append()- 在被选元素的结尾插入内容
prepend() - 在被选元素的开头插入内容
after() - 在被选元素之后插入内容
before() - 在被选元素之前插入内容
```

### 删除元素

```javascript
remove() - 删除被选元素（及其子元素）
empty() - 从被选元素中删除子元素
```

### 利用CSS方法改变属性

```javascript
addClass() - 向被选元素添加一个或多个类
removeClass() - 从被选元素删除一个或多个类
toggleClass() - 对被选元素进行添加/删除类的切换操作
css() 方法设置或返回被选元素的一个或多个样式属性。
```

## AJAX
Ajax即Asynchronous Javascript And XML（异步JavaScript和XML）在 2005年被Jesse James Garrett提出的新术语，用来描述一种使用现有技术集合的‘新’方法，包括: HTML 或 XHTML, CSS, JavaScript, DOM, XML, XSLT, 以及最重要的XMLHttpRequest。  使用Ajax技术网页应用能够快速地将增量更新呈现在用户界面上，而不需要重载（刷新）整个页面，这使得程序能够更快地回应用户的操作。
### load方法
load() 方法从服务器加载数据，并把返回的数据放入被选元素中。
语法：

```javascript
$(selector).load(URL,data,callback);
```

必需的 URL 参数规定您希望加载的 URL。
可选的 data 参数规定与请求一同发送的查询字符串键/值对集合。
可选的 callback 参数是 load() 方法完成后所执行的函数名称。
### get与post方法
GET - 从指定的资源请求数据
POST - 向指定的资源提交要处理的数据
语法：

```javascript
$.get(URL,callback);
```

必需的 URL 参数规定您希望请求的 URL。
可选的 callback 参数是请求成功后所执行的函数名。
$.post() 方法通过 HTTP POST 请求向服务器提交数据。

```javascript
$.post(URL,data,callback);
```

必需的 URL 参数规定您希望请求的 URL。
可选的 data 参数规定连同请求发送的数据。
可选的 callback 参数是请求成功后所执行的函数名。