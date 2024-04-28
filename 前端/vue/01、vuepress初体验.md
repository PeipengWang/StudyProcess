# 三分钟教你安装Vuepress

### II.**关于Vuepress**

​      首先介绍一下**Vuepress**，**VuePress** 是尤雨溪发布的一个基于 vue 的静态网站生成器，个人感觉如果用它搭建博客的话他比Hexo还简洁，如果能适应的话，加上一些主题还蛮好康的。

以Evan Xu大佬的博客为例（[Evan's blog (xugaoyi.com)](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fxugaoyi.com%2F&source=article&objectId=1942104)）



![img](https://raw.githubusercontent.com/PeipengWang/picture/master/040fbecb58dedf6566545acfe8614db6.png)

> Vuepress分为v0.x v1.x v2.x版本 ，本文以v1x版本作为例子，其余版本安装可能会与本教程略有差距

- Github：[GitHub - vuejs/vuepress: 📝 Minimalistic Vue-powered static site generator](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvuepress&source=article&objectId=1942104)
- Vuepress v1.x文档：[VuePress (vuejs.org)](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fv1.vuepress.vuejs.org%2Fzh%2F&source=article&objectId=1942104)

### III. 开始

> VuePress 需要Node.js＞= 8.6版本才行

1.咱们首先在你想安装的地方创建目录并进入

代码语言：javascript

```javascript
mkdir vuepress-starter && cd vuepress-starter
```

2.使用你觉得好的包管理器进行初始化

> 这里有yarn和npm两种方案，但是文档中有说明 如果你的现有项目依赖了 webpack 3.x，我们推荐使用 Yarn而不是 npm 来安装 VuePress。因为在这种情形下，npm 会生成错误的依赖树。 所以我们先以yarn为例

```javascript
yarn init
```

3.将 VuePress 安装为本地依赖

```javascript
yarn add -D vuepress
```

4.创建一个文档

```javascript
mkdir docs && echo '# Hello VuePress' > docs/README.md
```

5.在 `package.json` 中添加一些scripts

> 官方给的建议是添加。我的建议也是添加，不过具体还是得看您填不填加了

```javascript
{
  "scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
```



![注意逗号，有时候少了逗号会报错](https://raw.githubusercontent.com/PeipengWang/picture/master/02e64f1c21af11a3d0f1f16d0476ef49.png)

注意逗号，有时候少了逗号会报错

![官方文档](https://raw.githubusercontent.com/PeipengWang/picture/master/5b8bd0a7ef650aa5dd8515c302419f65.png)

6.在本地启动[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)

```javascript
yarn docs:dev
```

VuePress 会在[http://localhost:8080(opens new window)](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Flocalhost%3A8080%2F&source=article&objectId=1942104)启动一个热重载的开发服务器。

现在，你应该已经有了一个简单可用的 VuePress 文档。接下来，了解一下推荐的 [目录结构](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fdirectory-structure.html&source=article&objectId=1942104) 和 VuePress 中的 [基本配置](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fbasic-config.html&source=article&objectId=1942104)。

等你了解完上文介绍的基础概念，再去学习一下如何使用 [静态资源](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fassets.html&source=article&objectId=1942104)，[Markdown 拓展](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fmarkdown.html&source=article&objectId=1942104) 和 [在 Markdown 中使用 Vue](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fusing-vue.html&source=article&objectId=1942104) 来丰富你的文档内容。

当你的文档逐渐成型的时候，不要忘记 VuePress 的 [多语言支持](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fi18n.html&source=article&objectId=1942104) 并了解一下如何将你的文档 [部署](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fdeploy.html&source=article&objectId=1942104) 到任意静态文件服务器上。

### IV.注意事项

![npm版本为17，运行npm run docs:dev可能会报错](https://raw.githubusercontent.com/PeipengWang/picture/master/086ec3250c25810b0881c5ea592c45a3.png)

npm版本为17，运行npm run docs:dev可能会报错

# 目录结构

VuePress 遵循 **“约定优于配置”** 的原则，推荐的目录结构如下：

```
.
├── docs
│   ├── .vuepress (可选的)
│   │   ├── components (可选的)
│   │   ├── theme (可选的)
│   │   │   └── Layout.vue
│   │   ├── public (可选的)
│   │   ├── styles (可选的)
│   │   │   ├── index.styl
│   │   │   └── palette.styl
│   │   ├── templates (可选的, 谨慎配置)
│   │   │   ├── dev.html
│   │   │   └── ssr.html
│   │   ├── config.js (可选的)
│   │   └── enhanceApp.js (可选的)
│   │ 
│   ├── README.md
│   ├── guide
│   │   └── README.md
│   └── config.md
│ 
└── package.json
```

注意

请留意目录名的大写。

- `docs/.vuepress`: 用于存放全局的配置、组件、静态资源等。
- `docs/.vuepress/components`: 该目录中的 Vue 组件将会被自动注册为全局组件。
- `docs/.vuepress/theme`: 用于存放本地主题。
- `docs/.vuepress/styles`: 用于存放样式相关的文件。
- `docs/.vuepress/styles/index.styl`: 将会被自动应用的全局样式文件，会生成在最终的 CSS 文件结尾，具有比默认样式更高的优先级。
- `docs/.vuepress/styles/palette.styl`: 用于重写默认颜色常量，或者设置新的 stylus 颜色常量。
- `docs/.vuepress/public`: 静态资源目录。
- `docs/.vuepress/templates`: 存储 HTML 模板文件。
- `docs/.vuepress/templates/dev.html`: 用于开发环境的 HTML 模板文件。
- `docs/.vuepress/templates/ssr.html`: 构建时基于 Vue SSR 的 HTML 模板文件。
- `docs/.vuepress/config.js`: 配置文件的入口文件，也可以是 `YML` 或 `toml`。
- `docs/.vuepress/enhanceApp.js`: 客户端应用的增强。

注意

当你想要去自定义 `templates/ssr.html` 或 `templates/dev.html` 时，最好基于 [默认的模板文件 (opens new window)](https://github.com/vuejs/vuepress/blob/master/packages/@vuepress/core/lib/client/index.dev.html)来修改，否则可能会导致构建出错。



## 默认的页面路由

此处我们把 `docs` 目录作为 `targetDir` （参考 [命令行接口](https://vuepress.vuejs.org/zh/api/cli.html#基本用法)），下面所有的“文件的相对路径”都是相对于 `docs` 目录的。在项目根目录下的 `package.json` 中添加 `scripts` ：

```json
{
  "scripts": {
    "dev": "vuepress dev docs",
    "build": "vuepress build docs"
  }
}
```

对于上述的目录结构，默认页面路由地址如下：

| 文件的相对路径     | 页面路由地址   |
| ------------------ | -------------- |
| `/README.md`       | `/`            |
| `/guide/README.md` | `/guide/`      |
| `/config.md`       | `/config.html` |