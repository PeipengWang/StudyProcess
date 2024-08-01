## 1、安装git

## 2、建立互信

`ssh-keygen`命令，并按提示操作。你可以选择保留默认的文件名和路径（例如，`~/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`），或者指定一个自定义的文件名。

完成后，你的私钥将保存在`~/.ssh/id_rsa`（或你指定的文件名），公钥将保存在`~/.ssh/id_rsa.pub`。

```
scp ~/.ssh/id_rsa.pub root@47.121.29.164:~/.ssh/authorized_keys.tmp
cat ~/.ssh/authorized_keys.tmp >> ~/.ssh/authorized_keys  
rm ~/.ssh/authorized_keys.tmp
```

### 3、服务器搭建git库

```
cd ~  
mkdir sample  
cd sample  
git init --bare sample.git
```

这里，`git init --bare`命令创建了一个裸仓库（bare repository），这意味着仓库中只包含Git的数据结构（没有工作树）。这是因为你通常不会在服务器上直接修改代码；相反，你会在本地计算机上工作，然后将更改推送到服务器。

### 从本地计算机克隆仓库

```
git clone ssh://root@47.121.29.164:/root/sample.git
```

或者更常见的（因为SSH默认端口是22，所以通常可以省略端口号）：

```
git clone root@47.121.29.164:/root/sample.git
```

### 设置仓库的权限

在生产环境中，强烈建议不要使用`root`用户来运行Git仓库，因为这会增加安全风险。相反，你应该创建一个专用的用户（如`git`）来管理Git仓库。

确保你的Git仓库目录（在这个例子中是`/root/sample.git`）的权限设置得足够安全，以便只有授权的用户可以访问它。这通常意味着只有特定的用户（如`git`用户，而不是`root`）应该拥有对该目录的写入权限。但是，由于你正在使用`root`用户，这里我们暂时保持默认权限。