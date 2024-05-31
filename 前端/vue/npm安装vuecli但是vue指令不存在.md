

# npm安装vuecli但是vue指令不存在



## 前置条件

正常安装 Node.js 和 npm，如果你还没有安装 Node.js 和 npm，请从 [Node.js 官方网站](https://nodejs.org/) 下载并安装。安装 Node.js 时会同时安装 npm。

**全局安装 Vue CLI**： 打开终端并运行以下命令来全局安装 Vue CLI：

```shell
npm install -g @vue/cli
```

**验证安装**： 安装完成后，通过检查版本来验证 Vue CLI 是否安装正确：

```shell
vue --version
```

但是此时

```shell
vue --version
bash: vue: 未找到命令
```

## 原因与解决方法

这是因为安装的vue作为一个指令放到了nodejs的安装目录下，但是我在安装nodejs时将npm指令和node指令以软链接的方式来设置的，所以nodejs目录下的bin是无法获取到vue指令的

**排查方法**：

```shell
npm config get prefix
/home/node/node-v20.14.0-linux-x64
```

检查npm安装目录/home/node/node-v20.14.0-linux-x64的bin目录下有vue这个二进制文件

同时查看环境变量

```shell
printenv
```

发现环境变量不存在这个目录，因此需要将环境变量添加即可

```shell
vi .bashrc
export PATH=$PATH:/home/node/node-v20.14.0-linux-x64/bin
source .bashrc
```

**当然也可以将vue指令直接软链接**
