# iframe的页内与外部跳转
## iframe基础
<iframe> 标签规定一个内联框架。一个内联框架被用来在当前 HTML 文档中嵌入另一个文档。
所有的主流浏览器都支持<iframe>标签。你可以把提示的文字放到 <iframe> 和 </iframe>里面，这样不支持 <iframe>的浏览器就会出现提示的文字。
iframe 如何使用呢？

通常我们使用iframe直接在页面嵌套iframe标签指定的src就可以了。

比如：

```
    <!-- 
     <iframe> 标签规定一个内联框架
　　　这里写p 标签是为了看align的效果
     -->
<p style="overflow: hidden;">这是一些文本。 这是一些文本。 这是一些文本。这是一些文本。 这是一些文本。 这是一些文本。
<iframe name="myiframe" id="myrame" src="external_file.html" frameborder="0" align="left" width="200" height="200" scrolling="no">
<p>你的浏览器不支持iframe标签</p>
</iframe>
这是一些文本。 这是一些文本。 这是一些文本。这是一些文本。 这是一些文本。 这是一些文本。</p>
```
iframe 的常用属性：
　　 name ：  规定 <iframe> 的名称。
        width： 规定 <iframe> 的宽度。
        height ：规定 <iframe> 的高度。
        src ：规定在 <iframe> 中显示的文档的 URL。
        frameborder ： 规定是否显示 <iframe> 周围的边框。 (0为无边框，1位有边框)。
        align ：　 规定如何根据周围的元素来对齐 <iframe>。　(left,right,top,middle,bottom)。
        scrolling ： 规定是否在 <iframe> 中显示滚动条。 (yes,no,auto)
        sandbox属性如下
                 ""	启用所有限制条件
              allow-forms 允许表单提交。
              allow-same-origin 允许将内容作为普通来源对待。如果未使用该关键字，嵌入的内容将被视为一个独立的源。
              allow-scripts 允许脚本执行。
              allow-top-navigation	嵌入的页面的上下文可以导航（加载）内容到顶级的浏览上下文环境（browsing context）。如果未使用该关键字，这个操作将不可用。
那如何获取iframe里面的内容呢？
```
    var iframe = document.getElementById("myrame"); //获取iframe标签
    var iwindow = iframe.contentWindow; //获取iframe的window对象
    var idoc = iwindow.document; //获取iframe的document对象
    console.log(idoc.documentElement); //获取iframe的html
    console.log("body",idoc.body)
```


## top.location和window.location有什么区别？
window.location.href、location.href是本页面跳转，是在当前frame中打开新页
parent.location.href是上一层页面跳转
top.location.href是最外层的页面跳转，是在顶层frame中打开新页

