# Vue学习笔记

特点：

1、JavaScript框架

2、简化Dom操作

3、响应式的数据驱动

Vue2官网

https://v2.vuejs.org/v2/guide/

## Vue基础

### 第一个Vue程序

新建html文件：index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

<div id="app">
  {{ message }}
</div>
</body>
<script>
  var app = new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue!'
    }
  })
</script>
</html>
```

流程

1. 这个程序中引入作为依赖

```vue
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
```

2.将节点设置id

```vue
<div id="app">
```

3.利用el将数据挂载到节点的id上，message解释id中{{  }}中的数据，el实际上是限定了数据命中的范围

```vue
 el: '#app',
    data: {
      message: 'Hello Vue!'
    }
```

**el挂载点：**

vue会管理el选项命中元素及其内部的后代元素

也可以选择其他选择器，但是建议id选择器

可以使用其他双标签，但是不能使用HTML和BODY

**data数据对象**

1、字典数据对象

```json
     school:{
        name: "科技大学",
        mobile: "12306"
      },
```

模板中这样调用

```vue
  <h3>{{ school.name }} {{ school.mobile }}</h3>
```

2、数组对象

```json
 campus: ["南京校区", "成都校区", "济南校区"]
```

模板中这样调用

```vue
  <ul>
    <li>{{ campus[0] }}</li>
    <li>{{ campus[1] }}</li>
  </ul>
```

### 本地应用

#### v-text

v-text指令作用是：设置标签的内容（textContent）

默认写法会替换全部内容，使用差值表达式{{ }}可以替换指定内容

比如如下内容

```vue
    <h3 v-text="message"></h3>
    <h3 v-text="'学校：' + school.name"></h3>
    <h3 v-text="school.name">学校</h3> //这种写法“学校”会被删除
```

内部支持表达式

#### v-html

设置innerHtml标签

可以将html进行转码

实例

```
content: "<a href='http://www.peipengnote.cn'>笔记</a>"
    <p v-text="content"></p>
    <p v-html="content"></p>
```

分别展示如下，一个是原本的文本，一个是一个超级链接

```
<a href='http://www.peipengnote.cn'>笔记</a>
笔记
```

总结：

v-html指令的作用是：设置innerHtml标签

内容中有html的结构会被解析为标签

v-text指令无论内容是什么，只会解析为文本

解析文本使用v-text，需要解析html结构使用v-html

#### v-on

为元素绑定事件

事件类型：

click：点击事件

monseenter：离开事件

dbclick：双击事件

基本写法

```vue
    <input type="button" value="事件绑定" v-on:click="dolt">
    <input type="button" value="简写" @click="dolt">
```

写一个双击事件

```vue
    <input type="button" value="双击事件" @dblclick="eat_food">
```

methods里定义方法

```
methods: {
        dolt: function (){
            console.log("点击")
        },
          eat_food: function (){
            this.food = "好吃"
          }
      }
```

其中thid.food定义在data里

总结：

- v-on指令的作用是：为元素绑定事件

- 事件名不需要写on

- 指令可以简写为@
- 绑定的方法定义在methods属性中
- 方法内部可以通过this关键字可以访问定义在data中数据

#### 总结

上述代码总结为

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app2">
    <h3 v-text="message"></h3>
    <h3 v-text="'学校：' + school.name"></h3>
    <p v-text="content"></p>
    <p v-html="content"></p>
    <input type="button" value="事件绑定" v-on:click="dolt">
    <input type="button" value="简写" @click="dolt">
    <input type="button" value="双击事件" @dblclick="eat_food">
    <p>{{ food }}</p>
</div>
</body>
<script>
  var app = new Vue({
    el: '#app2',
    data: {
      message: 'You loaded this page on ' + new Date().toLocaleString(),
      school:{
        name: "科技大学",
        mobile: "12306"
      },
        content: "<a href='http://www.peipengnote.cn'>笔记</a>",
        food: ""
    },
      methods: {
        dolt: function (){
            console.log("点击")
        },
          eat_food: function (){
            this.food = "好吃"
          }
      }
  })
</script>
</html>
```

#### 计数器实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app2">

    <input class="input-num" type="button" value="-" v-on:click="dec"> <p v-text="num"></p> <input type="button" value="+" @click="add">

</div>
</body>
<script>
  var app = new Vue({
    el: '#app2',
    data: {
        num: 0
    },
      methods: {
        add: function (){
            if(this.num >= 10){
                return;
            }
            this.num = this.num+1;
        },
        dec: function (){
            if(this <= -10){
                return;
            }
            this.num = this.num-1;
        }
      }
  })
</script>
</html>
```



#### v-show

根据表达式的真假切换元素的显示和隐藏

代码:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app">
    <input type="button" value="切换显示" @click="showPict">
    <img src="logo.png" v-show="isShow">
</div>
</body>
<script>
  var app = new Vue({
    el: '#app',
    data: {
        isShow: true
    },
      methods:{
        showPict: function (){
            this.isShow = !this.isShow;
        }
      }
  })
</script>
</html>
```

- v-show指令的作用是：根据真假切换元素显示的状态
- 原理是修改元素的display，实时显示隐藏
- 指令后面的内容，最终都会解析为bool值
- 值为true元素显示，值为false元素隐藏
- 数据元素改变后，是否隐藏也会同步更新

#### v-if

根据表达式的真假，切换元素的显示和隐藏，操作的是dom元素

例如可以实现跟v-show完全一样的功能

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app">
    <input type="button" value="切换显示" @click="showPict">
    <img src="logo.png" v-show="isShow">
</div>
</body>
<script>
  var app = new Vue({
    el: '#app',
    data: {
        isShow: true
    },
      methods:{
        showPict: function (){
            this.isShow = !this.isShow;
        }
      }
  })
</script>
</html>
```



但是实际上，v-show只是将显示的内容设置为display: none;  但是v-if是直接将html标签也直接删掉

v-if也可以定义表达式来实现功能

先定义一个data

```
     data: {
            isShow: true,
            template: 40
        },
```

标签中设置表达式

```html
<p v-if="template > 35"> 热死啦</p>
```



此时，v-if会解析template是否大于35，大于35才会显示

总结：

- v-if作用：根据表达式的真假，切换元素的显示状态
- 本质上是通过操纵dom元素切换显示状态
- 表达式为true，元素存在于dom树中，为false则从dom树中删除
- 频繁切换则用v-show，反之则使用v-if，v-show的切换消耗更小。

#### v-bind

设置元素的属性

v-bind：属性=""

也可以直接省略

:属性=" "

实例

```html
 <img v-bind:src="imgSrc" >
 <img :src="imgSrc">
 <img :src="imgSrc" :class="{active:isActive}" v-on:click="showActive">
```

数据

```vue
<script>
    var app = new Vue({
        el: '#app',
        data: {
            isShow: true,
            template: 40,
            imgSrc: "logo.png",
            isActive: false
        },
        methods:{
            showPict: function (){
                this.isShow = !this.isShow;
            },
            showActive: function (){
                this.isActive = !this.isActive;
            }
        }
    })
</script>  
```



总结

- v-bind指令作用：为元素绑定属性
- 完整写法是v-bind:属性名
- v-bind可以省略为:属性名
- 需要动态的增删class建议使用对象的方式

#### 图片切换实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .active{
            border:1px solid red;
        }
    </style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app">

    <a href="#" v-show="index !== 0" v-on:click="pre" class="left">上一张</a>
    <img v-bind:src="imgArray[index]" >
    <a href="#" v-show="index !== 2" v-on:click="next" class="right">下一张</a>

</div>
</body>
<script>
    var app = new Vue({
        el: '#app',
        data: {
            imgArray: ["img.png", "img_1.png","img_2.png","img_3.png"],
            index: 0
        },
        methods:{
            next: function (){
                this.index++;
            },
            pre: function (){
              this.index--;
            }
        }
    })
</script>
</html>
```

#### v-for

根据数据生成列表结构

使用实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .active{
            border:1px solid red;
        }
    </style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app">
    <ul>
        <li v-for="(item,index) in arr">编号：{{ item }}  索引： {{ index }}</li>
    </ul>
<h3 v-for="item in vegetables">{{ item.name }}</h3>
    <button v-on:click="add">增加</button>
    <button v-on:click="remove">remove</button>
</div>
</body>
<script>
    var app = new Vue({
        el: '#app',
        data: {
            arr: ["1", "2","3","4"],
            vegetables: [
                {name: "西蓝花"},
                {name: "西红柿"}
            ]

        },
        methods:{
            add: function (){
                this.vegetables.push({name:"白菜"})
            },
            remove: function (){
                this.vegetables.pop()
            }
        }
    })
</script>
</html>
```

- v-for指令的作用：根据数据生成列表结构
- 数组经常和v-for结合使用
- 语法是（item， index）in 数据
- item和index可以结合其他指令一起使用
- 数组数据更新会同步到页面上

#### v-model

获取和设置表单元素的值（双向数据绑定）

实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .active{
            border:1px solid red;
        }
    </style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

<div id="app">
    <input type="button" value="修改" @click="setM(111)">
    <input type="text" v-model="message" @keyup.enter="getM">
    <p >{{ message }}</p>
</div>
</body>
<script>
    var app = new Vue({
        el: '#app',
        data:{
            message: "数据"
        },
        methods:{
            getM: function (){
                console.log(this.message)
            },
            setM: function (p){
                this.message = p
            }
        }
    })
</script>
</html>
```

- v-mode指令作用是便捷的设置和获取表单元素的值
- 绑定的数据会和表单元素值相关联
- 绑定的数据《--》表单元素的值

#### 记事本功能

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>


<div id="app">

    <input v-model="message" @keyup.enter="add(message)">
    <ul>
        <li v-for="(item,index) in listMessage" > {{index}}. {{ item }}
        </li>
    </ul>
    <input type="button" value="删除" @click="remove()">
    <input type="button" value="清空" @click="removeAll()">
    <p>总数：{{  listMessage.length }}</p>
</div>
</body>
<script>
    var app = new Vue({
        el: '#app',
        data: {
            listMessage: ["吃饭", "睡觉"],
            message: ""
        },
        methods:{
            add: function (){
                this.listMessage.push(this.message)
            },
            remove: function (){
                this.listMessage.pop()
            },
            removeAll: function (){
                this.listMessage = []
            }
        }
    })
</script>
</html>
```



