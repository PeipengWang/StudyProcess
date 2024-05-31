# Vue 组件常用关键字



## 模板语法

```html
<template>
  <div>{{ message }}</div>
</template>
```

## 脚本

```javascript
<script>
export default {
  data() {
    return {
      message: 'Hello Vue!'
    }
  }
}
</script>
```

## 样式

```css
<style scoped>
div {
  color: red;
}
</style>

```

## **指令**

- **v-bind**: 绑定属性

```html
<img v-bind:src="imageSrc">
```

- **v-model**: 双向绑定

```html
<input v-model="inputValue">
```

- **v-if**: 条件渲染

  ```html
  <p v-if="seen">Now you see me</p>
  ```

- **v-for**: 列表渲染

```html
<ul>
  <li v-for="item in items" :key="item.id">{{ item.text }}</li>
</ul>
```

- **v-on**: 事件绑定

```html
<button v-on:click="doSomething">Click me</button>
```

## 生命周期钩子

**created**: 实例创建完成时调用

```javascript
created() {
  console.log('Component created')
}
```



**mounted**: 实例挂载到 DOM 时调用

```javascript
mounted() {
  console.log('Component mounted')
}
```

**updated**: 数据更新时调用

```javascript
destroyed() {
  console.log('Component destroyed')
}
```

**destroyed**: 实例销毁时调用

```javascript
destroyed() {
  console.log('Component destroyed')
}
```



