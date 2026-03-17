

目标是 **部署到微信公众号（H5）**

其实就是把项目 **编译为 H5 页面，然后配置到公众号菜单**。

完整流程。

------

# 一、运行项目前准备

需要安装这些环境：

| 软件      | 作用                |
| --------- | ------------------- |
| Node.js   | 运行前端            |
| HBuilderX | uni-app官方开发工具 |
| Git       | 下载代码            |

推荐使用：

- **HBuilderX**

下载
https://www.dcloud.io/hbuilderx.html

------

# 二、下载项目

```bash
git clone https://github.com/ruoyi-mobile/ruoyi-app.git
```

或者直接下载 ZIP。

------

# 三、用 HBuilderX 打开项目

步骤：

1. 打开 **HBuilderX**
2. 点击

```
文件 → 打开目录
```

选择：

```
ruoyi-app
```

项目结构大致：

```
ruoyi-app
 ├ api
 ├ components
 ├ pages
 ├ store
 ├ static
 ├ utils
 ├ App.vue
 ├ main.js
 └ pages.json
```

------

# 四、配置后端接口地址

编辑：

```
utils/request.js
```

找到：

```javascript
baseUrl: 'http://localhost:8080'
```

修改为你的后端地址：

```javascript
baseUrl: 'http://你的服务器IP:8080'
```

如果部署线上：

```javascript
baseUrl: 'https://api.xxx.com'
```

------

# 五、本地运行

在 **HBuilderX** 点击：

```
运行 → 运行到浏览器 → Chrome
```

系统会自动：

```
编译 H5
启动本地服务器
```

浏览器访问：

```
http://localhost:8080
```

即可看到移动端页面。

### 注意二开需要重新生成 AppID

1️⃣ 打开项目文件：

```
manifest.json
```

2️⃣ 找到：

```
DCloud appid
```

3️⃣ 点击：

```
重新获取AppID
```

或者：

```
重新生成
```

4️⃣ HBuilderX 会自动生成类似：

```
__UNI__XXXXXXX
```

新的 AppID。

保存即可。

------

# 六、打包为 H5

部署公众号必须先打包 H5。

在 HBuilderX：

```
发行 → 网站-H5手机版
```

编译完成后生成：

```
/unpackage/dist/build/h5
```

里面是：

```
index.html
static
assets
```

------

# 七、部署到服务器

把 `h5` 目录上传到服务器：

例如：

```
/www/wwwroot/mobile
```

用 **Nginx** 配置：

```
server {
    listen 80;
    server_name mobile.xxx.com;

    location / {
        root /www/wwwroot/mobile;
        index index.html;
    }
}
```

访问：

```
http://mobile.xxx.com
```

即可看到页面。

------

# 八、接入微信公众号

**账号准备**：

- 注册微信小程序账号（[微信公众平台](https://mp.weixin.qq.com/)），并完成小程序的基本配置（如 AppID、名称、类目）
- 记录小程序的 **AppID**（开发设置中可查，不要用测试号）

微信公众号菜单配置：

进入：

```
公众号后台
自定义菜单
```

新增菜单：

```
工作台
```

菜单链接：

```
https://mobile.xxx.com
```

用户点击菜单即可打开 H5 页面。



### 打包并发布到微信小程序

#### 步骤 1：在 HBuilderX 中打包成微信小程序代码

1. 打开 HBuilderX，选中你的 UniApp 项目；
2. 点击顶部菜单栏「发行」→「小程序 - 微信」；
3. 选择「发行」（开发阶段可先选「运行」，直接打开微信开发者工具预览）；
4. 等待打包完成，HBuilderX 会提示「已生成微信小程序项目」，并显示代码输出路径（如 `unpackage/dist/dev/mp-weixin`）。

#### 步骤 2：在微信开发者工具中上传代码

1. 打开微信开发者工具，点击「导入」，选择上述打包后的 `mp-weixin` 文件夹，输入小程序 AppID；
2. 导入后，先在左侧模拟器中测试功能是否正常（如接口请求、页面跳转、样式适配）；
3. 测试无误后，点击右上角「上传」，填写「版本号」（如 1.0.0）和「备注」（如「应急能力统计功能上线」），确认上传。

#### 步骤 3：在微信公众平台提交审核并发布

1. 登录 [微信公众平台](https://mp.weixin.qq.com/)，进入「版本管理」→「开发版本」；
2. 找到你刚上传的代码版本，点击「提交审核」，按提示填写审核信息（如小程序类目、功能说明）；
3. 等待微信审核（一般 1-3 个工作日），审核通过后，在「发布版本」中点击「发布」，小程序即可上线。

------

# 九、公众号 H5 需要注意

微信要求：

### 必须 HTTPS

必须使用：

```
https://
```

不能用：

```
http://
```

------

### 配置安全域名

公众号后台：

```
设置
公众号设置
功能设置
JS接口安全域名
```

添加：

```
mobile.xxx.com
```

------

# 十、最终架构

完整运行结构：

```
微信公众号
     │
     │ 点击菜单
     ▼
uniapp H5页面
     │
     ▼
RuoYi 后端 API
     │
     ▼
MySQL
```

------

# 十一、如果想做真正的微信小程序

uniapp还可以：

```
发行 → 小程序 → 微信小程序
```

生成小程序代码，然后用：

**WeChat Developer Tools**

上传即可。

------

# 十二、很多人会踩的坑

1️⃣ 跨域问题
2️⃣ HTTPS证书
3️⃣ 后端接口地址
4️⃣ 微信域名白名单

------

✅ 如果你需要，我可以再帮你：

- **画一张 RuoYi + uniapp + 微信公众号完整架构图**
- **给你一套最完整部署方案（服务器 + Nginx + HTTPS）**
- **教你 30 分钟把 RuoYi 移动端跑起来并接入公众号**。