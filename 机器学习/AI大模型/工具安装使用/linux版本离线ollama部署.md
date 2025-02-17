#### 离线安装

步骤1：查看服务器CPU的型号

```
## 查看Linux系统CPU型号命令，我的服务器cpu型号是x86_64
lscpu
```

![image-20250217194926795](https://raw.githubusercontent.com/PeipengWang/picture/master/examimage-20250217194926795.png)

步骤2：根据CPU型号下载Ollama[安装包](https://so.csdn.net/so/search?q=安装包&spm=1001.2101.3001.7020)，并保存到`/home/Ollama`目录

下载地址 https://github.com/ollama/ollama/releases/

```
# x86_64 CPU选择下载ollama-linux-amd64
# aarch64|arm64 CPU选择下载ollama-linux-arm64
```



步骤3：离线下载Linux环境的Ollama安装脚本，并保存到`/home/Ollama`目录

```
## 下载地址1，浏览器中打开下面地址
https://ollama.com/install.sh

## 下载地址2
https://github.com/ollama/ollama/blob/main/scripts/install.sh
## 国内加速可访问https://gitee.com/ai-big-model/ollama/blob/main/scripts/install.sh
```

第一处修改，注释下载链接

```
status "Downloading ollama..."
## 在install.sh的第65行
#curl --fail --show-error --location --progress-bar -o $TEMP_DIR/ollama "https://ollama.com/download/ollama-linux-${ARCH}${VER_PARAM}"
```

第二处修改，修改ollama**安装目录**

```
status "Installing ollama to $BINDIR..."
$SUDO install -o0 -g0 -m755 -d $BINDIR
## 在install.sh的第73行
#$SUDO install -o0 -g0 -m755 $TEMP_DIR/ollama $BINDIR/ollama
$SUDO install -o0 -g0 -m755 ./ollama-linux-amd64  $BINDIR/ollama
```

修改后的install.[sh文件](https://so.csdn.net/so/search?q=sh文件&spm=1001.2101.3001.7020)，v0.1.31版本

步骤4：运行 install.sh脚本 ,安装

```
# 执行installl.sh脚本，需要sudo 权限  chmod +x install.sh
./install.sh
# 如果报错误权限不足，执行
chmod +x install.sh
# 如果报错误： bash: ./build_android.sh：/bin/sh^M：解释器错误: 没有那个文件或目录，执行
sed -i 's/\r$//' install.sh
```

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/exam749a3189e1af7524e409932306289c66.png)

步骤5：配置大模型下载目录

```
# 执行命令
vim ~/.bashrc
# 配置 OLLAMA_MODELS 环境变量自定义路径
### ollama model dir 改为自定义的路径，默认路径/usr/share/ollama/.ollama/models 
export OLLAMA_MODELS=/home/Ollama/ollama_cache
# 复制/usr/share/ollama/.ollama/models目录中（blobs  manifests）的文件夹到OLLAMA_MODELS环境变量目录
cp -r /usr/share/ollama/.ollama/models /home/Ollama/ollama_cache
```

步骤6：运行大模型，如通义千问

```
# 需要先将大模型下载到OLLAMA_MODELS文件中
# ollama run <模型名称>
ollama run qwen
```

步骤7：关闭 Ollama 服务

```
# 关闭ollama服务
service ollama stop
```



1. 配置ollma
   有一些额外的环境变量需要配置，常用的是监听host、下载路径（因为是离线，用处不大）、多cuda使用

2. ```
   vim /etc/systemd/system/ollama.service
   ```

   ![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/examfb148e37e9764af79d420fc18cf30775.png)

这里host设置主要是为了后面open webui，可能无法访问。
cuda设置，主要是优先使用3号gpu。

```
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="CUDA_VISIBLE_DEVICES=3,2"
Environment="OLLAMA_MODELS=/data/ollama/model"
```

注意，这里会出现一个问题，如果不给`OLLAMA_MODELS`的文件夹777权限，启动可能可能会失败。

```
sudo chmod 777 /XXX
```

顺便记录一下，uninstall的链接：https://github.com/ollama/ollama/blob/main/docs/linux.md#uninstall
