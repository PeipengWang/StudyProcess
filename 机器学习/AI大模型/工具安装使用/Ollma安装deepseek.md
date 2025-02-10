### 一：安装Ollama

如果想要在本地运行 DeepSeek 需要用到 Ollama 这个工具，这是一个开源的本地大模型运行工具。

我们可以访问 https://ollama.com/ 进入 Ollama 官网下载 Ollama ，下载时有三个系统的安装包可选择，这里只需要选择下载我们电脑对应的操作系统版本即可，这里我选择的是 Windows 版本。

![DeepSeek本地部署，再也不怕服务器崩了！](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexi679eb6f9a2aef522.png_e1080.jpg)



Ollama 安装包下载完成后直接双击安装即可，安装速度还是比较快的。

Ollama 安装完成后需要打开电脑的 CMD ，也就是命令提示符，只需要在电脑下方的搜索框中输入 cmd 即可打开。

打开后在命令提示符窗口中输入 ollama help 并按回车键，这个操作是为了验证这个软件是否安装成功，如果没有报错的话则表示这个软件安装成功。

### 二：下载部署 Deepseek 模型

回到 https://ollama.com/ 网址中，在网页上方搜索框中输入 Deepseek-r1，这个 Deepseek-r1 就是我们需要本地部署的一个模型。

点击 Deepseek-r1 后会进入详情界面，里面有多个参数规模可供选择，从 1.5b 到 671b 都有。

![image-20250210100714046](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexiimage-20250210100714046.png)

需注意的是，这里我们需要根据自己电脑的硬件配置来选择模型大小，下面是一个模型大小配置参考表格，大家可根据自己的电脑配置来自行选择，当然了，部署的本地模型越大，使用的深度求索效果就越好。

![image-20250210100815409](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexiimage-20250210100815409.png)

选择好模型规模后，复制右边的一个命令。

命令复制完成后回到命令提示符窗口，将刚刚复制的命令粘贴到命令提示符窗口中并按回车键即可下载模型

![image-20250210100503363](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexiimage-20250210100503363.png)

模型下载完成后，我们就直接可以在命令提示符面板中使用它了。

以后如果我们需要再次使用 Deepseek 这个模型的话，我们可以直接打开命令提示符窗口，只需要再次在命令符提示窗口中输入上面复制的指令即可。



### 三：可视化图文交互界面 Chatbox

虽然我们可以在本地正常使用 Deepseek 这个模型了，但是这个 AI 工具的面板是非常简陋的，很多人使用不习惯，这时我们就可以通过 Chatbox 这个可视化图文交互界面来使用它。

点击 https://chatboxai.app/zh 进入 Chatbox 官网，Chatbox 虽然有本地客户端，但我们也可以直接使用网页版。

进入 Chatbox 网页版本后点击使用自己的 API Key 或本地模型

![DeepSeek本地部署，再也不怕服务器崩了！](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexi679eb73448a611703.png_e1080.jpg)

点击后会进入模型提供方选择界面，这里选择 Ollama API 。

![DeepSeek本地部署，再也不怕服务器崩了！](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexi679eb70122ca51623.png_e1080.jpg)

这里需要注意的是，为了能够 Ollama 能远程链接，这里我们最好看一下 Chatbox 提供的教程，根据这个教程操作一下。

![DeepSeek本地部署，再也不怕服务器崩了！](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexi679eb702139195993.png_e1080.jpg)

这个教程还是非常简单的，如果是 Windows 操作系统则只需要配置一下环境变量即可，配置完环境变量后需重启一下 Ollama 程序。

在 Windows 上，Ollama 会继承你的用户和系统环境变量。

1. 通过任务栏退出 Ollama。

2. 打开设置（Windows 11）或控制面板（Windows 10），并搜索“环境变量”。

3. 点击编辑你账户的环境变量。

   为你的用户账户编辑或创建新的变量 **OLLAMA_HOST**，值为 **0.0.0.0**； 为你的用户账户编辑或创建新的变量 **OLLAMA_ORIGINS**，值为 *****。

4. 点击确定/应用以保存设置。

5. 从 Windows 开始菜单启动 Ollama 应用程序。

重启 Ollama 程序后，我们需要将 Chatbox 设置界面关闭并重新打开，重新打开 Chatbox 设置界面后即可选择 Deepseek 模型了，选择后 Deepseek 模型后点击保存即可。

![DeepSeek本地部署，再也不怕服务器崩了！](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexi679eb70b9e7128614.png_e1080.jpg)

接下来只需要在 Chatbox 中新建对话即可使用 Deepseek 模型了，以下图为例，上方是它的思考过程，下方是它给出的答案。

![DeepSeek本地部署，再也不怕服务器崩了！](https://raw.githubusercontent.com/PeipengWang/picture/master/jiqixuexi679eb70d65fb08322.png_e1080.jpg)
