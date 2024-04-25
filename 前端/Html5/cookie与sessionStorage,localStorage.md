# cookie
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/76372917f8ae4e5892ef6681f214a86c.png)![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/76233a901b9d4aa2b5f71b3cf34ffa83.png)


localStorage 和 sessionStorage 属性允许在浏览器中存储 key/value 对的数据。
会话Cookie:保存在浏览器内存中，关闭浏览器，Cookie便被销毁了。
普通Cookie：保存在硬盘上，设置了过期时间，过期后销毁。

sessionStorage,localStorage,Cookie都放在客户端浏览器，区别在于
sessionStorage，localStorage存放的参数，不会主动传递给服务器端，解决了无用参数传递的问题。
sessionStorage和localStorage比较，sessionStorage存放在浏览器内存，关闭浏览器后就销毁了；localStorage存放在硬盘，到达有效期后销毁。
## sessionStorage
定义和使用
localStorage 和 sessionStorage 属性允许在浏览器中存储 key/value 对的数据。
sessionStorage 用于临时保存同一窗口(或标签页)的数据，在关闭窗口或标签页之后将会删除这些数据。
提示: 如果你想在浏览器窗口关闭后还保留数据，可以使用 localStorage 属性， 该数据对象没有过期时间，今天、下周、明年都能用，除非你手动去删除。
语法
window.sessionStorage
保存数据语法：
sessionStorage.setItem("key", "value");
读取数据语法：
var lastname = sessionStorage.getItem("key");
删除指定键的数据语法：
sessionStorage.removeItem("key");
删除所有数据：
sessionStorage.clear();
## localStorage
定义和使用
localStorage 和 sessionStorage 属性允许在浏览器中存储 key/value 对的数据。
localStorage 用于长久保存整个网站的数据，保存的数据没有过期时间，直到手动去删除。
localStorage 属性是只读的。
提示: 如果你只想将数据保存在当前会话中，可以使用 sessionStorage 属性， 该数据对象临时保存同一窗口(或标签页)的数据，在关闭窗口或标签页之后将会删除这些数据。


### localStorage 的优势
 1、localStorage 拓展了 cookie 的 4K 限制。
 2、localStorage 会可以将第一次请求的数据直接存储到本地，这个相当于一个 5M 大小的针对于前端页面的数据库，相比于 cookie 可以节约带宽，但是这个却是只有在高版本的浏览器中才支持的。
### localStorage 的局限
 1、浏览器的大小不统一，并且在 IE8 以上的 IE 版本才支持 localStorage 这个属性。
 2、目前所有的浏览器中都会把localStorage的值类型限定为string类型，这个在对我们日常比较常见的JSON对象类型需要一些转换。
 3、localStorage在浏览器的隐私模式下面是不可读取的。
 4、localStorage本质上是对字符串的读取，如果存储内容多的话会消耗内存空间，会导致页面变卡。
 5、localStorage不能被爬虫抓取到。
localStorage 与 sessionStorage 的唯一一点区别就是 localStorage 属于永久性存储，而 sessionStorage 属于当会话结束的时候，sessionStorage 中的键值对会被清空。
### localStorage 写入
首先在使用 localStorage 的时候，我们需要判断浏览器是否支持 localStorage 这个属性：
```
if(! window.localStorage){
    alert("浏览器不支持localstorage");
    return false;
}else{
    //主逻辑业务
}
localStorage 的写入有三种方法:
if(！window.localStorage){
    alert("浏览器不支持localstorage");
    return false;
}else{
    var storage=window.localStorage;
    //写入a字段
    storage["a"]=1;
    //写入b字段
    storage.b=1;
    //写入c字段
    storage.setItem("c",3);
    console.log(typeof storage["a"]);
    console.log(typeof storage["b"]);
    console.log(typeof storage["c"]);
}
```
这里要特别说明一下 localStorage 的使用也是遵循同源策略的，所以不同的网站直接是不能共用相同的 localStorage。
刚刚存储进去的是 int 类型，但是打印出来却是 string 类型，这个与 localStorage 本身的特点有关，localStorage 只支持 string 类型的存储。
### localStorage 的读取
```
if(!window.localStorage){
    alert("浏览器不支持localstorage");
}else{
    var storage=window.localStorage;
    //写入a字段
    storage["a"]=1;
    //写入b字段
    storage.b=1;
    //写入c字段
    storage.setItem("c",3);
    console.log(typeof storage["a"]);
    console.log(typeof storage["b"]);
    console.log(typeof storage["c"]);
    //第一种方法读取
    var a=storage.a;
    console.log(a);
    //第二种方法读取
    var b=storage["b"];
    console.log(b);
    //第三种方法读取
    var c=storage.getItem("c");
    console.log(c);
}
```
这里面是三种对 localStorage 的读取，其中官方推荐的是 getItem\setItem 这两种方法对其进行存取，不要问我这个为什么，因为这个我也不知道。
我之前说过localStorage就是相当于一个前端的数据库的东西，数据库主要是增删查改这四个步骤，这里的读取和写入就相当于增、查的这两个步骤。
下面我们就来说一说 localStorage 的删、改这两个步骤。
改这个步骤比较好理解，思路跟重新更改全局变量的值一样，这里我们就以一个为例来简单的说明一下：
```
if(!window.localStorage){
    alert("浏览器不支持localstorage");
}else{
var storage=window.localStorage;
    //写入a字段
    storage["a"]=1;
    //写入b字段
    storage.b=1;
    //写入c字段
    storage.setItem("c",3);
    console.log(storage.a);
    // console.log(typeof storage["a"]);
    // console.log(typeof storage["b"]);
    // console.log(typeof storage["c"]);
    /*分割线*/
    storage.a=4;
    console.log(storage.a);
}
```
这个在控制台上面我们就可以看到已经 a 键已经被更改为 4 了。
localStorage 的删除
1、将localStorage的所有内容清除
```
var storage=window.localStorage;
storage.a=1;
storage.setItem("c",3);
console.log(storage);
storage.clear();
console.log(storage);
```
2、 将 localStorage 中的某个键值对删除
```
var storage=window.localStorage;
storage.a=1;
storage.setItem("c",3);
console.log(storage);
storage.removeItem("a");
console.log(storage.a);
```
控制台查看结果:
localStorage 键的获取
```
var storage=window.localStorage;
storage.a=1;
storage.setItem("c",3);
for(var i=0;i<storage.length;i++){
    var key=storage.key(i);
    console.log(key);
}
```
使用 key() 方法，向其中出入索引即可获取对应的键。
localStorage 其他注意事项
一般我们会将 JSON 存入 localStorage 中，但是在 localStorage 会自动将 localStorage 转换成为字符串形式。
这个时候我们可以使用 JSON.stringify() 这个方法，来将 JSON 转换成为 JSON 字符串。
实例：
```
if(!window.localStorage){
    alert("浏览器不支持localstorage");
}else{
    var storage=window.localStorage;
    var data={
        name:'xiecanyong',
        sex:'man',
        hobby:'program'
    };
    var d=JSON.stringify(data);
    storage.setItem("data",d);
    console.log(storage.data);
}
```
读取之后要将 JSON 字符串转换成为 JSON 对象，使用 JSON.parse() 方法：
```
var storage=window.localStorage;
var data={
    name:'xiecanyong',
    sex:'man',
    hobby:'program'
};
var d=JSON.stringify(data);
storage.setItem("data",d);
```
//将JSON字符串转换成为JSON对象输出
var json=storage.getItem("data");
var jsonObj=JSON.parse(json);
console.log(typeof jsonObj);
打印出来是 Object 对象。
另外还有一点要注意的是，其他类型读取出来也要进行转换。
### localstorage 存储 json 格式的方法
比如有 json 数据如下：
```
var data = {
    test:"text",
    test1:"123456",
    test2:"字段值",
}
```
存储方式为如下，将 json 转化为字符串再存入:
```
localStorage.setItem("data", JSON.stringify(data));
```
而使用方法则是如下：
```
var data = localStorage.getItem("data");
data = JSON.parse(data);
```
取出存储的字段，再重新转化为 json 形式。
以上对于 json 数据同样有效，所以对于一些接口请求的数据可以通过这种方式本地化。