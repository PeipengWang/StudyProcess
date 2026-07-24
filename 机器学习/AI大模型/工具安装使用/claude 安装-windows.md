## 安装方法1：PowerShell 带代理安装（最推荐，最快）

把下面整条复制进 PowerShell（改端口为你本地代理，常见 7890/10809）：

```
$env:HTTP_PROXY="http://127.0.0.1:7890"
$env:HTTPS_PROXY="http://127.0.0.1:7890"
irm https://claude.ai/install.ps1 | iex
```

- 原理：让安装脚本走代理去下 Google 上的二进制
- 成功标志：最后输出 `Claude Code installed successfully`

------

## 安装方法二：用 winget 安装（不用翻墙，国内也能成功）

PowerShell 直接运行：

```
winget install Anthropic.ClaudeCode
```

- 官方原生包，**不走 [storage.googleapis.com](https://storage.googleapis.com)**
- 自动加 PATH，装完直接：

```
claude --version
```

## 配置环境变量--deepseek

```
# 1. 设置代理地址（DeepSeek 兼容 Claude 接口）
$env:ANTHROPIC_BASE_URL = "https://api.deepseek.com/anthropic"

# 2. 设置你的 DeepSeek API Key（替换成你自己的 sk-xxx）
$env:ANTHROPIC_AUTH_TOKEN = "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# 3. 指定模型
$env:ANTHROPIC_MODEL = "deepseek-v4-pro"
```

