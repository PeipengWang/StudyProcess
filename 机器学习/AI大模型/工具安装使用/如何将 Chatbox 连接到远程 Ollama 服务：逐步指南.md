## 安装 Ollama

Ollama 是一个开源的本地模型运行工具，可以方便地下载和运行各种开源模型，比如 Llama、Qwen、DeepSeek 等。这个工具支持 Windows、MacOS、Linux 等操作系统。

[Ollama 下载地址](https://ollama.com/)

## 下载并运行本地模型

下载并安装 Ollama 后，请打开命令行终端，输入命令下载并运行本地模型。你可以在这里查看到所有 ollama 支持的模型列表：[Ollama 模型列表](https://ollama.com/models)。

举例1：下载并运行 llama3.2 模型

```bash
ollama run llama3.2
```

举例2：下载并运行 deepseek-r1:8b 模型（注意：Ollama 上的 DeepSeek R1 模型实际上是蒸馏模型）

```bash
ollama run deepseek-r1:8b
```

## 在 Chatbox 中连接本地 Ollama 服务

在 Chatbox 中打开设置，在模型提供方中选择 Ollama，即可在模型下拉框中看见你运行的本地模型。

![Ollama Chatbox 教程](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexiollama_guide_3.png)

点击保存，即可正常聊天使用。

## 在 Chatbox 中连接远程 Ollama 服务

除了可以轻松连接本地 Ollama 服务，Chatbox 也支持连接到运行在其他机器上的远程 Ollama 服务。

例如，你可以在家中的电脑上运行 Ollama 服务，并在手机或其他电脑上使用 Chatbox 客户端连接到这个服务。

你需要确保远程 Ollama 服务正确配置并暴露在当前网络中，以便 Chatbox 可以访问。**默认情况下，需要对远程 Ollama 服务进行简单的配置**。

### 如何配置远程 Ollama 服务？

默认情况下，Ollama 服务仅在本地运行，不对外提供服务。要使 Ollama 服务能够对外提供服务，你需要设置以下两个环境变量：

```
OLLAMA_HOST=0.0.0.0
OLLAMA_ORIGINS=*
```

### 在 MacOS 上配置

1. 打开命令行终端，输入以下命令：

   ```bash
   launchctl setenv OLLAMA_HOST "0.0.0.0"
   launchctl setenv OLLAMA_ORIGINS "*"
   ```

2. 重启 Ollama 应用，使配置生效。

### 在 Windows 上配置

在 Windows 上，Ollama 会继承你的用户和系统环境变量。

1. 通过任务栏退出 Ollama。

2. 打开设置（Windows 11）或控制面板（Windows 10），并搜索“环境变量”。

3. 点击编辑你账户的环境变量。

   为你的用户账户编辑或创建新的变量 **OLLAMA_HOST**，值为 **0.0.0.0**； 为你的用户账户编辑或创建新的变量 **OLLAMA_ORIGINS**，值为 *****。

4. 点击确定/应用以保存设置。

5. 从 Windows 开始菜单启动 Ollama 应用程序。

### 在 Linux 上配置

如果 Ollama 作为 systemd 服务运行，应使用 systemctl 设置环境变量：

1. 调用 `systemctl edit ollama.service` 编辑 systemd 服务配置。这将打开一个编辑器。

2. 在 [Service] 部分下为每个环境变量添加一行 Environment：

   ```
   [Service]
   Environment="OLLAMA_HOST=0.0.0.0"
   Environment="OLLAMA_ORIGINS=*"
   ```

3. 保存并退出。

4. 重新加载 systemd 并重启 Ollama：

   ```
   systemctl daemon-reload
   systemctl restart ollama
   ```

### 服务 IP 地址

配置后，Ollama 服务将能在当前网络（如家庭 Wifi）中提供服务。你可以使用其他设备上的 Chatbox 客户端连接到此服务。

Ollama 服务的 IP 地址是你电脑在当前网络中的地址，通常形式如下：

```
192.168.XX.XX
```

在 Chatbox 中，将 API Host 设置为：

```
http://192.168.XX.XX:11434
```

### 注意事项

- 可能需要在防火墙中允许 Ollama 服务的端口（默认为 11434），具体取决于你的操作系统和网络环境。
- 为避免安全风险，请不要将 Ollama 服务暴露在公共网络中。家庭 Wifi 网络是一个相对安全的环境。

