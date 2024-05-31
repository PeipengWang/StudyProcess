# Vue开发：搭建开发环境

## 安装node环境

## 设置镜像

1.npm设置镜像

```shell
npm config set registry=镜像地址
```

//设置阿里镜像

```shell
npm config set registry=https://registry.npm.taobao.org
```

2.npm取消镜像

```shell
npm config delete registry
```

3.npm查看镜像信息

```shell
npm config get registry
```

## 设置代理

 1.设置http和https代理

```shell
//设置http代理
npm config set proxy = http://代理服务器地址:端口号
//设置https代理
npm config set https-proxy https://代理服务器地址:端口号
```

2.取消代理

```shell
npm config delete proxy
npm config delete https-proxy
```

3.代理用户名和密码设置

如果代理需要认证的话，可以使用如下方式设置

```shell
npm config set proxy http://username:password@server:port
npm confit set https-proxy http://username:password@server:port
```

4.查看代理信息

```shell
npm config list
```

## 安装vue-cli工具

首先，我们使用配置一全局安装vue-cli。vue-cli是vue.js官方脚手架工具，提供了基于node的从单元测试、系统测试到打包发布的完整环境。

```
npm install -g vue-cli
```

确保vue-cli安装成功后，将npm环境切换到配置二，因为vue-cli要下载webpack模板文件

```shell
vue create hello-world
```



## Idea直接安装

需要环境

node.js

idea插件：vue.js

新建项目--》选择JavaScript--》选择vue.js--》确定

## 运行

```shell
cd hello-world
 npm run serve
> hello-world@0.1.0 serve
> vue-cli-service serve
 INFO  Starting development server...
 DONE  Compiled successfully in 5653ms                                                                                                                                                                       
  App running at:
  - Local:   http://localhost:8080/ 
  - Network: http://ip:8080/

  Note that the development build is not optimized.
  To create a production build, run npm run build.


```

直接访问即可
