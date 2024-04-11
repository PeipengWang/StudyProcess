# vue3
1、Hello Vue（文本）
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Vue hello</title>
  <script src="https://unpkg.com/vue@next"></script>
</head>
<body>
<div id="hello-vue" class="demo">
  {{ message }}
</div>

<script>
  const HelloVueApp = {
    data() {
      return {
        message: 'Hello Vue!!'
      }
    }
  }
  Vue.createApp(HelloVueApp).mount('#hello-vue')

</script>
</body>
</html>
```
mount('#hello-vue') 将 Vue 应用 HelloVueApp 挂载到 <div id="hello-vue"></div> 中。
{{ }} 用于输出对象属性和函数返回值。
{{ message }} 对应应用中 message 的值。
data 选项
data 选项是一个函数。Vue 在创建新组件实例的过程中调用此函数。它应该返回一个对象，然后 Vue 会通过响应性系统将其包裹起来，并以 $data 的形式存储在组件实例中
2、Html
使用 v-html 指令用于输出 html 代码：
