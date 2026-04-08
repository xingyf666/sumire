# AI 模型应用

## 供应商

### Ollama

#### 安装路径

下载 [Ollama](https://ollama.com/) 用于安装大模型

```shel
irm https://ollama.com/install.ps1 | iex
```

默认会安装在 C 盘，最好下载安装包，然后执行命令行来修改安装路径

```powershell
.\OllamaSetup.exe /DIR=F:\Ollama\App
```

打开程序后，在设置中修改模型的安装路径。还需要在系统环境变量中添加

```
CUDA_VISIBLE_DEVICES GPU-5ddfde31-da8b-2c86-7f86-74bcf6c73c02
OLLAMA_GPU_LAYER cuda
OLLAMA_MODELS F:\Ollama\Models
```

> >有时会出现无法调用 GPU 运行，导致卡顿的问题，这时候最直接的方法就是重装。



#### 安装模型

点击 Models 搜索想要安装的大模型

![](image-20260311102522817.png)

复制模型名称，执行

```shell
ollama run qwen3.5:9b
```



#### 运行开销

在 ollama 执行时，输入命令查看模型开销

```shell
PS F:\Ollama\App> ollama ps
NAME          ID              SIZE     PROCESSOR          CONTEXT    UNTIL      
qwen3.5:9b    6488c96fa5fa    10 GB    47%/53% CPU/GPU    32768      4 minutes from now
```

可以看到 CPU/GPU 占比。

> >注意不要使用过大的上下文容量，否则会很卡顿。



### OpenRouter

OpenRouter 可以提供大量模型的 API 。创建 API key 后，记录下生成的 key，它只显示一次，后续无法查看

```
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
```



### Deepseek

类似地，deepseek 的 API 同理

```
sk-15cae4f8e5ef4433b5282fb46b29669b
```



## 本地 Agent

Agent 首先通过大模型思考解决方案，然后调用外部工具完成任务，例如 Cline 就是一个 Agent 。



### Cline

将 key 放入设置栏，然后在 OpenRouter 网站中找到一个免费的模型

![](image-20260311175414551.png)



在 `mcp.so` 中搜索 `fetch` 服务，在右侧找到配置代码

![](image-20260311180830705.png)

在 Cline 中配置 MCP Servers，将代码复制到生成的 `json` 文件中

![](image-20260311180944679.png)

> >需要等待服务下载完成以后才可以使用。



然后提供一个爬取网页的任务（选一个没有反爬虫机制的网站）

![](image-20260311181035587.png)



### Claude

打开管理员终端，输入

```shell
irm https://claude.ai/install.ps1 | iex
```

安装后还需要修改 `User` 目录下的 `.claude.json` 文件，添加 `"hasCompletedOnboarding": true` 来跳过登录验证

```json
{
  "installMethod": "native",
  "autoUpdates": false,
  "firstStartTime": "2026-03-12T17:44:36.880Z",
  "opusProMigrationComplete": true,
  "sonnet1m45MigrationComplete": true,
  "userID": "d85939a048ee49d52c2650802b045564da265077cef3a222799af173d714b99b",
  "changelogLastFetched": 1773337477523,
  "autoUpdatesProtectedForNative": true,
  "hasCompletedOnboarding": true
}
```

安装 CC switch 用于切换国产模型

```embed
title: "CC switch"
image: ""
description: "Claude Code、Codex、Gemini CLI、OpenCode 和 OpenClaw 的全方位管理工具"
url: "https://github.com/farion1231/cc-switch/blob/v3.12.0/README_ZH.md"
```

关于 CC switch 中的配置，参考 MiniMax 的官方文档

```embed
title: "通过 AI 编程工具接入 - MiniMax 开放平台文档中心"
image: "https://filecdn.minimax.chat/public/58eca777-e31f-448a-9823-e2220e49b426.png"
description: "MiniMax-M2.5 & MiniMax-M2.5-highspeed 兼容 OpenAI 和 Anthropic 接口协议，适用于代码助手、Agent 工具、AI IDE 等多种场景。"
url: "https://platform.minimaxi.com/docs/guides/text-ai-coding-tools"
```

注意请求地址 `https://api.minimaxi.com/anthropic` 与官网地址格式不同

![](image-20260313035746132.png)



### Codex

Codex 是 OpenAI 推出的开源 Agent，执行如下命令安装

```shell
npm i -g @openai/codex
```



### OpenCode

需要安装 `nodejs`，使用 scoop 安装

```shell
scoop install nodejs
```

然后运行官网的安装命令，需要管理员权限

```shell
npm i -g opencode-ai
```



## Claw

### OpenClaw

#### 基础配置

安装 OpenClaw 需要 Node 高版本，参考这篇帖子

```embed
title: "?tl=zh-hans"
image: "https://www.reddit.com/favicon.ico"
description: ""
url: "https://www.reddit.com/r/node/comments/sot1ep/whats_the_best_way_to_install_the_newest_npm_and/?tl=zh-hans"
```

使用 Volta 维护 Node

```embed
title: "Getting Started | Volta"
image: "https://docs.volta.sh/assets/wordmark.jpg"
description: "Getting Started"
url: "https://docs.volta.sh/guide/getting-started"
```

安装 Volta 然后再安装 Node

```shell
curl https://get.volta.sh | bash
volta install node
```

然后可以安装

```shell
npm install -g openclaw@latest
```

还需要使用 brew 安装包

```shell
bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

如果有网络问题，重试几次就可以，然后按照它提示的步骤将 brew 添加到环境变量。



#### 配置 ClawHub

脚本安装

```shell
npm i -g clawhub
```

然后可以使用 ClawHub 安装各类 skills 。



### ZeroClaw

通过 scoop 安装

```shell
scoop install main/zeroclaw
```



#### 基础配置

使用如下命令开启配置引导

```shell
zeroclaw onboard
```



#### 配置 QQ

```embed
title: "zeroclaw/docs/reference/api/channels-reference.md at master · zeroclaw-labs/zeroclaw"
image: "https://opengraph.githubassets.com/5752cd42fd1a5a89a51e29b8686517dd480a9a086d45d51238f17c9b63cb92f3/zeroclaw-labs/zeroclaw"
description: "Fast, small, and fully autonomous AI personal assistant infrastructure, ANY OS, ANY PLATFORM — deploy anywhere, swap anything 🦀 - zeroclaw-labs/zeroclaw"
url: "https://github.com/zeroclaw-labs/zeroclaw/blob/master/docs/reference/api/channels-reference.md"
```

可以单独配置 QQ，然后直接退出，再执行第二行命令才能连接 QQ

```shell
zeroclaw onboard --channels-only
zeroclaw daemon
```



### Nanobot

Nanobot 是轻量级的 OpenClaw

```embed
title: "GitHub - HKUDS/nanobot: ”🐈 nanobot: The Ultra-Lightweight OpenClaw”"
image: "https://opengraph.githubassets.com/4ef3189caf1dd1684e0c6b1889d0883d2f0ad80b3e09f3857bb9d11beb7a0d1a/HKUDS/nanobot"
description: "″🐈 nanobot: The Ultra-Lightweight OpenClaw”. Contribute to HKUDS/nanobot development by creating an account on GitHub."
url: "https://github.com/HKUDS/nanobot"
```

使用 uv 快速安装

```shell
uv tool install nanobot-ai
```



#### 基础配置

配置工作空间、模型和提供商

```json
"agents": {
  "defaults": {
    "workspace": "F:/nanobot",
    "model": "MiniMax-M2.5",
    "provider": "minimax",
    "maxTokens": 8192,
    "temperature": 0.1,
    "maxToolIterations": 40,
    "memoryWindow": 100,
    "reasoningEffort": null
  }
},
```

执行初始化，然后开启对话

```shell
nanobot onboard
nanobot agent
```



#### 配置 QQ

配置 QQ 机器人

```json
"qq": {
  "enabled": true,
  "appId": "1903512300",
  "secret": "klnpsvz38DJPWemv4EOl9XwLlBc4WzSw",
  "allowFrom": ["3352072018B256C9A754625E52BEBD60"],
  "msgFormat": "markdown"
},
```

其中 `appId, secret` 通过 QQ 开发平台获得，`allowFrom` 最开始设为 `["*"]` 允许任何人访问

```embed
title: "QQ开放平台"
image: "https://qqminiapp.cdn-go.cn/qq-open-platform/a16966ce/favicon.svg"
description: "QQ小程序是连接年轻用户的新方式，覆盖8亿新生代活跃网民。轻便快捷的开发模式，将能在QQ内被轻松获取和传播"
url: "https://q.qq.com/#/apps"
```

配置好后开启网关即可在 QQ 中聊天

```shell
nanobot gateway
```

在对话记录中找到自己的 `id` 填入 `allowFrom` 即可。



#### 配置 Email

参考如下格式填写，其中 `imapPassword, smtpPassword` 要从邮箱的 IMAP 协议管理中获得

```json
"email": {
  "enabled": true,
  "consentGranted": true,
  "imapHost": "imap.qq.com",
  "imapPort": 993,
  "imapUsername": "814083398@qq.com",
  "imapPassword": "usjjqzdfgtwnbdjc",
  "imapMailbox": "INBOX",
  "imapUseSsl": true,
  "smtpHost": "smtp.qq.com",
  "smtpPort": 587,
  "smtpUsername": "814083398@qq.com",
  "smtpPassword": "usjjqzdfgtwnbdjc",
  "smtpUseTls": true,
  "smtpUseSsl": false,
  "fromAddress": "814083398@qq.com",
  "autoReplyEnabled": true,
  "pollIntervalSeconds": 30,
  "markSeen": true,
  "maxBodyChars": 12000,
  "subjectPrefix": "Re: ",
  "allowFrom": ["*"]
},
```

在“设置-账户”中找到“管理服务”，进入页面后可以点击生成授权码

![](image-20260322172808879.png)



#### 网络搜索

使用 `duckduckgo` 作为网络搜索工具

```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "duckduckgo"
      }
    }
  }
}
```



### AstrBot

AstrBot 对国内接口支持比较好，关键是开发者较多

```embed
title: "GitHub - AstrBotDevs/AstrBot: Agentic IM Chatbot infrastructure that integrates lots of IM platforms, LLMs, plugins and AI feature, and can be your openclaw alternative. ✨"
image: "https://opengraph.githubassets.com/ff960012c21ac55da4a4b56d37cf5b440ea1a78b383eb5955f27a607f45e2c34/AstrBotDevs/AstrBot"
description: "Agentic IM Chatbot infrastructure that integrates lots of IM platforms, LLMs, plugins and AI feature, and can be your openclaw alternative. ✨ - AstrBotDevs/AstrBot"
url: "https://github.com/AstrBotDevs/AstrBot"
```

似乎没有 `x64` 版本的客户端，而 `uv` 直接部署占用 C 盘空间，因此直接本地部署

```shell
git clone https://github.com/AstrBotDevs/AstrBot.git
cd AstrBot
uv sync
uv run main.py
```

参考 MiniMax 的 OpenAI 接口配置

```embed
title: "OpenAI API 兼容 - MiniMax 开放平台文档中心"
image: "https://filecdn.minimax.chat/public/58eca777-e31f-448a-9823-e2220e49b426.png"
description: "通过 OpenAI SDK 调用 MiniMax 模型"
url: "https://platform.minimaxi.com/docs/api-reference/text-openai-api"
```

填入 OpenAI 商源，然后自定义模型 `MiniMax-M2.5`，创建 QQSocket

> >注意 QQ 机器人不能删除，白名单也不能删除，一定不要添加白名单。



### HermitClaw

可自主学习研究的小螃蟹，提供一个小房间可视化 AI 的工作状态

```embed
title: "GitHub - brendanhogan/hermitclaw"
image: "https://opengraph.githubassets.com/9748862dc2022e9ed8ba72cdf79d01cfa7253bc5eb033e85fcaef4b5dbf3649c/brendanhogan/hermitclaw"
description: "Contribute to brendanhogan/hermitclaw development by creating an account on GitHub."
url: "https://github.com/brendanhogan/hermitclaw"
```



## 大模型原理

### MCP

MCP (Model Context Protocol) 即模型上下文协议，使用 Cline 作为中间代理的 MCP 服务流程如下

![](image-20260311174816894.png)



#### 获取天气

安装 MCP 开发所需的两个依赖

```shell
uv init weather
uv venv
uv add "mcp[cli]" httpx
```

然后提供一个 `weather.py` 文件

```python
from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather", log_level="ERROR")


NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"


async def make_nws_request(url: str) -> dict[str, Any] | None:
    """Make a request to the NWS API with proper error handling."""
    headers = {"User-Agent": USER_AGENT, "Accept": "application/geo+json"}
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers=headers, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except Exception:
            return None


def format_alert(feature: dict) -> str:
    """Format an alert feature into a readable string."""
    props = feature["properties"]
    return f"""
Event: {props.get("event", "Unknown")}
Area: {props.get("areaDesc", "Unknown")}
Severity: {props.get("severity", "Unknown")}
Description: {props.get("description", "No description available")}
Instruction: {props.get("instruction", "No specific instructions provided")}
    """


@mcp.tool()
async def get_alerts(state: str) -> str:
    """Get weather alerts for a US state.

    Args:
        state: Two-letter US state code (e.g. CA, NY)
    """
    url = f"{NWS_API_BASE}/alerts/active/area/{state}"
    data = await make_nws_request(url)

    if not data or "features" not in data:
        return "Could not retrieve alerts for the specified state."

    if not data["features"]:
        return "No active alerts for the specified state."

    alerts = [format_alert(feature) for feature in data["features"]]
    return "\n---\n".join(alerts)


@mcp.tool()
async def get_forecast(lat: float, lon: float) -> str:
    """Get weather forecast for a location.

    Args:
        lat: Latitude of the location
        lon: Longitude of the location
    """
    url = f"{NWS_API_BASE}/points/{lat},{lon}"
    data = await make_nws_request(url)

    if not data or "properties" not in data or "forecast" not in data["properties"]:
        return "Could not retrieve forecast for the specified location."

    forecast_url = data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    if (
        not forecast_data
        or "properties" not in forecast_data
        or "periods" not in forecast_data["properties"]
    ):
        return "Could not retrieve forecast details for the specified location."

    periods = forecast_data["properties"]["periods"]
    formatted_forecast = "\n---\n".join(
        f"{period['name']}: {period['detailedForecast']}" for period in periods
    )
    return formatted_forecast


if __name__ == "__main__":
    mcp.run(transport="stdio")

```

在 `json` 配置文件中提供该 MCP 服务的命令

```json
{
  "mcpServers": {
    "weather": {
      "disabled": false,
      "timeout": 60,
      "command": "uv",
      "args": [
        "--directory",
        "E:/Desktop/learn/uv/weather",
        "run",
        "weather.py"
      ],
      "transportType": "stdio",
      "autoApprove": [
        "get_alerts",
        "get_forecast"
      ]
    }
  }
}
```

注意加载完成后勾选启动所有工具

![](image-20260312210733222.png)

> >也可以使用 Cline 可以直接让其提供一个 MCP 服务并调用。



#### 截取 MCP 交互

可以在中间添加一个 `mcp_logger.py` 来获取 Cline 与 MCP 服务之间的交互记录

```python
import sys
import subprocess
import threading
import argparse
import os

# --- Configuration ---
LOG_FILE = os.path.join(os.path.dirname(os.path.realpath(__file__)), "mcp_io.log")
# --- End Configuration ---

# --- Argument Parsing ---
parser = argparse.ArgumentParser(
    description="Wrap a command, passing STDIN/STDOUT verbatim while logging them.",
    usage="%(prog)s <command> [args...]",
)
# Capture the command and all subsequent arguments
parser.add_argument(
    "command",
    nargs=argparse.REMAINDER,
    help="The command and its arguments to execute.",
)

open(LOG_FILE, "w", encoding="utf-8")

if len(sys.argv) == 1:
    parser.print_help(sys.stderr)
    sys.exit(1)

args = parser.parse_args()

if not args.command:
    print("Error: No command provided.", file=sys.stderr)
    parser.print_help(sys.stderr)
    sys.exit(1)

target_command = args.command
# --- End Argument Parsing ---

# --- I/O Forwarding Functions ---
# These will run in separate threads


def forward_and_log_stdin(proxy_stdin, target_stdin, log_file):
    """Reads from proxy's stdin, logs it, writes to target's stdin."""
    try:
        while True:
            # Read line by line from the script's actual stdin
            line_bytes = proxy_stdin.readline()
            if not line_bytes:  # EOF reached
                break

            # Decode for logging (assuming UTF-8, adjust if needed)
            try:
                line_str = line_bytes.decode("utf-8")
            except UnicodeDecodeError:
                line_str = (
                    f"[Non-UTF8 data, {len(line_bytes)} bytes]\n"  # Log representation
                )

            # Log with prefix
            log_file.write(f"输入: {line_str}")
            log_file.flush()  # Ensure log is written promptly

            # Write the original bytes to the target process's stdin
            target_stdin.write(line_bytes)
            target_stdin.flush()  # Ensure target receives it promptly

    except Exception as e:
        # Log errors happening during forwarding
        try:
            log_file.write(f"!!! STDIN Forwarding Error: {e}\n")
            log_file.flush()
        except Exception:
            pass  # Avoid errors trying to log errors if log file is broken

    finally:
        # Important: Close the target's stdin when proxy's stdin closes
        # This signals EOF to the target process (like test.sh's read loop)
        try:
            target_stdin.close()
            log_file.write("--- STDIN stream closed to target ---\n")
            log_file.flush()
        except Exception as e:
            try:
                log_file.write(f"!!! Error closing target STDIN: {e}\n")
                log_file.flush()
            except Exception:
                pass


def forward_and_log_stdout(target_stdout, proxy_stdout, log_file):
    """Reads from target's stdout, logs it, writes to proxy's stdout."""
    try:
        while True:
            # Read line by line from the target process's stdout
            line_bytes = target_stdout.readline()
            if not line_bytes:  # EOF reached (process exited or closed stdout)
                break

            # Decode for logging
            try:
                line_str = line_bytes.decode("utf-8")
            except UnicodeDecodeError:
                line_str = f"[Non-UTF8 data, {len(line_bytes)} bytes]\n"

            # Log with prefix
            log_file.write(f"输出: {line_str}")
            log_file.flush()

            # Write the original bytes to the script's actual stdout
            proxy_stdout.write(line_bytes)
            proxy_stdout.flush()  # Ensure output is seen promptly

    except Exception as e:
        try:
            log_file.write(f"!!! STDOUT Forwarding Error: {e}\n")
            log_file.flush()
        except Exception:
            pass
    finally:
        try:
            log_file.flush()
        except Exception:
            pass
        # Don't close proxy_stdout (sys.stdout) here


# --- Main Execution ---
process = None
log_f = None
exit_code = 1  # Default exit code in case of early failure

try:
    # Open log file in append mode ('a') for the threads
    log_f = open(LOG_FILE, "a", encoding="utf-8")

    # Start the target process
    # We use pipes for stdin/stdout
    # We work with bytes (bufsize=0 for unbuffered binary, readline() still works)
    # stderr=subprocess.PIPE could be added to capture stderr too if needed.
    process = subprocess.Popen(
        target_command,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,  # Capture stderr too, good practice
        bufsize=0,  # Use 0 for unbuffered binary I/O
    )

    # Pass binary streams to threads
    stdin_thread = threading.Thread(
        target=forward_and_log_stdin,
        args=(sys.stdin.buffer, process.stdin, log_f),
        daemon=True,  # Allows main thread to exit even if this is stuck (e.g., waiting on stdin) - reconsider if explicit join is needed
    )

    stdout_thread = threading.Thread(
        target=forward_and_log_stdout,
        args=(process.stdout, sys.stdout.buffer, log_f),
        daemon=True,
    )

    # Optional: Handle stderr similarly (log and pass through)
    stderr_thread = threading.Thread(
        target=forward_and_log_stdout,  # Can reuse the function
        args=(process.stderr, sys.stderr.buffer, log_f),  # Pass stderr streams
        # Add a different prefix in the function if needed, or modify function
        # For now, it will log with "STDOUT:" prefix - might want to change function
        # Let's modify the function slightly for this
        daemon=True,
    )

    # A slightly modified version for stderr logging
    def forward_and_log_stderr(target_stderr, proxy_stderr, log_file):
        """Reads from target's stderr, logs it, writes to proxy's stderr."""
        try:
            while True:
                line_bytes = target_stderr.readline()
                if not line_bytes:
                    break
                try:
                    line_str = line_bytes.decode("utf-8")
                except UnicodeDecodeError:
                    line_str = f"[Non-UTF8 data, {len(line_bytes)} bytes]\n"
                log_file.write(f"STDERR: {line_str}")  # Use STDERR prefix
                log_file.flush()
                proxy_stderr.write(line_bytes)
                proxy_stderr.flush()
        except Exception as e:
            try:
                log_file.write(f"!!! STDERR Forwarding Error: {e}\n")
                log_file.flush()
            except Exception:
                pass
        finally:
            try:
                log_file.flush()
            except Exception:
                pass

    stderr_thread = threading.Thread(
        target=forward_and_log_stderr,
        args=(process.stderr, sys.stderr.buffer, log_f),
        daemon=True,
    )

    # Start the forwarding threads
    stdin_thread.start()
    stdout_thread.start()
    stderr_thread.start()  # Start stderr thread too

    # Wait for the target process to complete
    process.wait()
    exit_code = process.returncode

    # Wait briefly for I/O threads to finish flushing last messages
    # Since they are daemons, they might exit abruptly with the main thread.
    # Joining them ensures cleaner shutdown and logging.
    # We need to make sure the pipes are closed so the reads terminate.
    # process.wait() ensures target process is dead, pipes should close naturally.
    stdin_thread.join(timeout=1.0)  # Add timeout in case thread hangs
    stdout_thread.join(timeout=1.0)
    stderr_thread.join(timeout=1.0)


except Exception as e:
    print(f"MCP Logger Error: {e}", file=sys.stderr)
    # Try to log the error too
    if log_f and not log_f.closed:
        try:
            log_f.write(f"!!! MCP Logger Main Error: {e}\n")
            log_f.flush()
        except Exception:
            pass  # Ignore errors during final logging attempt
    exit_code = 1  # Indicate logger failure

finally:
    # Ensure the process is terminated if it's still running (e.g., if logger crashed)
    if process and process.poll() is None:
        try:
            process.terminate()
            process.wait(timeout=1.0)  # Give it a moment to terminate
        except Exception:
            pass  # Ignore errors during cleanup
        if process.poll() is None:  # Still running?
            try:
                process.kill()  # Force kill
            except Exception:
                pass  # Ignore kill errors

    # Final log message
    if log_f and not log_f.closed:
        try:
            log_f.close()
        except Exception:
            pass  # Ignore errors during final logging attempt

    # Exit with the target process's exit code
    sys.exit(exit_code)

```

同时还要修改 `json` 配置，加入 `mcp_logger.py` 作为执行文件

```json
{
  "mcpServers": {
    "weather": {
      "disabled": false,
      "timeout": 60,
      "command": "python",
      "args": [
        "E:/Desktop/learn/uv/weather/mcp_logger.py",
        "uv",
        "--directory",
        "E:/Desktop/learn/uv/weather",
        "run",
        "weather.py"
      ],
      "transportType": "stdio",
      "autoApprove": [
        "get_alerts",
        "get_forecast"
      ]
    }
  }
}
```

让 Cline 执行 MCP 服务，就可以在 `mcp_io.log` 中看到交互过程

- “输入”是 Cline 发送的内容
- “输出”是 MCP 服务发送的内容

例如其中一段 MCP 服务的内容为

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "tools": [
            {
                "name": "get_alerts",
                "description": "Get weather alerts for a US state.\n\n    Args:\n        state: Two-letter US state code (e.g. CA, NY)\n    ",
                "inputSchema": {
                    "properties": {
                        "state": {
                            "title": "State",
                            "type": "string"
                        }
                    },
                    "required": [
                        "state"
                    ],
                    "title": "get_alertsArguments",
                    "type": "object"
                },
                "outputSchema": {
                    "properties": {
                        "result": {
                            "title": "Result",
                            "type": "string"
                        }
                    },
                    "required": [
                        "result"
                    ],
                    "title": "get_alertsOutput",
                    "type": "object"
                }
            },
            {
                "name": "get_forecast",
                "description": "Get weather forecast for a location.\n\n    Args:\n        lat: Latitude of the location\n        lon: Longitude of the location\n    ",
                "inputSchema": {
                    "properties": {
                        "lat": {
                            "title": "Lat",
                            "type": "number"
                        },
                        "lon": {
                            "title": "Lon",
                            "type": "number"
                        }
                    },
                    "required": [
                        "lat",
                        "lon"
                    ],
                    "title": "get_forecastArguments",
                    "type": "object"
                },
                "outputSchema": {
                    "properties": {
                        "result": {
                            "title": "Result",
                            "type": "string"
                        }
                    },
                    "required": [
                        "result"
                    ],
                    "title": "get_forecastOutput",
                    "type": "object"
                }
            }
        ]
    }
}
```

其中

- `inputSchema` 规定了函数的输入格式，包括变量名、类型
- `outputSchema` 规定了函数的返回格式

> >事实上，可以在命令行中手动执行 `json` 中的命令，然后复制 Cline 发送的消息，代替 Cline 与 MCP 服务进行交互。



#### Function Calling

Function Calling 是模型本身具备的函数调用能力，具有如下特征

1. 解析工具列表
2. 挑选工具并给出参数
3. 解析工具执行结果

而 MCP 则提供一套函数调用协议，规定将外部工具的名称、效果、调用方式输入到模型中的规范。



#### 聚合网站

可以在如下几个网站搜索现成的 MCP 服务

```embed
title: "精选2025优质MCP服务器_全球MCP Server集合平台 | AIBase"
image: "https://mcp.aibase.cn/apple-touch-icon.png"
description: "发现最全面的MCP Server(Model Context Protocol服务器)集合，包括Google Drive、GitHub、Slack等热门集成。MCP作为‘AI应用的USB-C端口’，让AI模型与外部数据源和工具建立安全标准化连接，提升您的AI能力和工作效率"
url: "https://mcp.aibase.cn/"
```

```embed
title: "MCP Server（MCP 服务器）"
image: "https://mcp.so/logo.png"
description: "最大的 MCP Server（MCP 服务器）集合，包括优秀的 MCP Server（MCP 服务器）和 Claude MCP 集成。搜索和发现 MCP Server（MCP 服务器）以增强您的 AI 能力。"
url: "https://mcp.so/zh"
```

```embed
title: "MCP Market"
image: ""
description: "优秀的 MCP 服务器和客户端目录，用于将 AI 代理连接到您喜爱的工具。"
url: "https://mcpmarket.com/zh"
```

```embed
title: "Smithery"
image: ""
description: "The fastest way to extend your AI"
url: "https://smithery.ai/"
```

```embed
title: "Awesome MCP Servers"
image: "https://mcpservers.org/api/og"
description: "A curated collection of MCP (Model Context Protocol) servers. Search and discover MCP servers and clients."
url: "https://mcpservers.org/zh-CN/"
```



### 模型交互协议

#### 截取模型交互

可以在中间添加一个 `llm_logger.py` 来获取 Cline 与大模型之间的交互记录

```python
import sys
import subprocess
import threading
import argparse
import os

# --- Configuration ---
LOG_FILE = os.path.join(os.path.dirname(os.path.realpath(__file__)), "mcp_io.log")
# --- End Configuration ---

# --- Argument Parsing ---
parser = argparse.ArgumentParser(
    description="Wrap a command, passing STDIN/STDOUT verbatim while logging them.",
    usage="%(prog)s <command> [args...]",
)
# Capture the command and all subsequent arguments
parser.add_argument(
    "command",
    nargs=argparse.REMAINDER,
    help="The command and its arguments to execute.",
)

open(LOG_FILE, "w", encoding="utf-8")

if len(sys.argv) == 1:
    parser.print_help(sys.stderr)
    sys.exit(1)

args = parser.parse_args()

if not args.command:
    print("Error: No command provided.", file=sys.stderr)
    parser.print_help(sys.stderr)
    sys.exit(1)

target_command = args.command
# --- End Argument Parsing ---

# --- I/O Forwarding Functions ---
# These will run in separate threads


def forward_and_log_stdin(proxy_stdin, target_stdin, log_file):
    """Reads from proxy's stdin, logs it, writes to target's stdin."""
    try:
        while True:
            # Read line by line from the script's actual stdin
            line_bytes = proxy_stdin.readline()
            if not line_bytes:  # EOF reached
                break

            # Decode for logging (assuming UTF-8, adjust if needed)
            try:
                line_str = line_bytes.decode("utf-8")
            except UnicodeDecodeError:
                line_str = (
                    f"[Non-UTF8 data, {len(line_bytes)} bytes]\n"  # Log representation
                )

            # Log with prefix
            log_file.write(f"输入: {line_str}")
            log_file.flush()  # Ensure log is written promptly

            # Write the original bytes to the target process's stdin
            target_stdin.write(line_bytes)
            target_stdin.flush()  # Ensure target receives it promptly

    except Exception as e:
        # Log errors happening during forwarding
        try:
            log_file.write(f"!!! STDIN Forwarding Error: {e}\n")
            log_file.flush()
        except Exception:
            pass  # Avoid errors trying to log errors if log file is broken

    finally:
        # Important: Close the target's stdin when proxy's stdin closes
        # This signals EOF to the target process (like test.sh's read loop)
        try:
            target_stdin.close()
            log_file.write("--- STDIN stream closed to target ---\n")
            log_file.flush()
        except Exception as e:
            try:
                log_file.write(f"!!! Error closing target STDIN: {e}\n")
                log_file.flush()
            except Exception:
                pass


def forward_and_log_stdout(target_stdout, proxy_stdout, log_file):
    """Reads from target's stdout, logs it, writes to proxy's stdout."""
    try:
        while True:
            # Read line by line from the target process's stdout
            line_bytes = target_stdout.readline()
            if not line_bytes:  # EOF reached (process exited or closed stdout)
                break

            # Decode for logging
            try:
                line_str = line_bytes.decode("utf-8")
            except UnicodeDecodeError:
                line_str = f"[Non-UTF8 data, {len(line_bytes)} bytes]\n"

            # Log with prefix
            log_file.write(f"输出: {line_str}")
            log_file.flush()

            # Write the original bytes to the script's actual stdout
            proxy_stdout.write(line_bytes)
            proxy_stdout.flush()  # Ensure output is seen promptly

    except Exception as e:
        try:
            log_file.write(f"!!! STDOUT Forwarding Error: {e}\n")
            log_file.flush()
        except Exception:
            pass
    finally:
        try:
            log_file.flush()
        except Exception:
            pass
        # Don't close proxy_stdout (sys.stdout) here


# --- Main Execution ---
process = None
log_f = None
exit_code = 1  # Default exit code in case of early failure

try:
    # Open log file in append mode ('a') for the threads
    log_f = open(LOG_FILE, "a", encoding="utf-8")

    # Start the target process
    # We use pipes for stdin/stdout
    # We work with bytes (bufsize=0 for unbuffered binary, readline() still works)
    # stderr=subprocess.PIPE could be added to capture stderr too if needed.
    process = subprocess.Popen(
        target_command,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,  # Capture stderr too, good practice
        bufsize=0,  # Use 0 for unbuffered binary I/O
    )

    # Pass binary streams to threads
    stdin_thread = threading.Thread(
        target=forward_and_log_stdin,
        args=(sys.stdin.buffer, process.stdin, log_f),
        daemon=True,  # Allows main thread to exit even if this is stuck (e.g., waiting on stdin) - reconsider if explicit join is needed
    )

    stdout_thread = threading.Thread(
        target=forward_and_log_stdout,
        args=(process.stdout, sys.stdout.buffer, log_f),
        daemon=True,
    )

    # Optional: Handle stderr similarly (log and pass through)
    stderr_thread = threading.Thread(
        target=forward_and_log_stdout,  # Can reuse the function
        args=(process.stderr, sys.stderr.buffer, log_f),  # Pass stderr streams
        # Add a different prefix in the function if needed, or modify function
        # For now, it will log with "STDOUT:" prefix - might want to change function
        # Let's modify the function slightly for this
        daemon=True,
    )

    # A slightly modified version for stderr logging
    def forward_and_log_stderr(target_stderr, proxy_stderr, log_file):
        """Reads from target's stderr, logs it, writes to proxy's stderr."""
        try:
            while True:
                line_bytes = target_stderr.readline()
                if not line_bytes:
                    break
                try:
                    line_str = line_bytes.decode("utf-8")
                except UnicodeDecodeError:
                    line_str = f"[Non-UTF8 data, {len(line_bytes)} bytes]\n"
                log_file.write(f"STDERR: {line_str}")  # Use STDERR prefix
                log_file.flush()
                proxy_stderr.write(line_bytes)
                proxy_stderr.flush()
        except Exception as e:
            try:
                log_file.write(f"!!! STDERR Forwarding Error: {e}\n")
                log_file.flush()
            except Exception:
                pass
        finally:
            try:
                log_file.flush()
            except Exception:
                pass

    stderr_thread = threading.Thread(
        target=forward_and_log_stderr,
        args=(process.stderr, sys.stderr.buffer, log_f),
        daemon=True,
    )

    # Start the forwarding threads
    stdin_thread.start()
    stdout_thread.start()
    stderr_thread.start()  # Start stderr thread too

    # Wait for the target process to complete
    process.wait()
    exit_code = process.returncode

    # Wait briefly for I/O threads to finish flushing last messages
    # Since they are daemons, they might exit abruptly with the main thread.
    # Joining them ensures cleaner shutdown and logging.
    # We need to make sure the pipes are closed so the reads terminate.
    # process.wait() ensures target process is dead, pipes should close naturally.
    stdin_thread.join(timeout=1.0)  # Add timeout in case thread hangs
    stdout_thread.join(timeout=1.0)
    stderr_thread.join(timeout=1.0)


except Exception as e:
    print(f"MCP Logger Error: {e}", file=sys.stderr)
    # Try to log the error too
    if log_f and not log_f.closed:
        try:
            log_f.write(f"!!! MCP Logger Main Error: {e}\n")
            log_f.flush()
        except Exception:
            pass  # Ignore errors during final logging attempt
    exit_code = 1  # Indicate logger failure

finally:
    # Ensure the process is terminated if it's still running (e.g., if logger crashed)
    if process and process.poll() is None:
        try:
            process.terminate()
            process.wait(timeout=1.0)  # Give it a moment to terminate
        except Exception:
            pass  # Ignore errors during cleanup
        if process.poll() is None:  # Still running?
            try:
                process.kill()  # Force kill
            except Exception:
                pass  # Ignore kill errors

    # Final log message
    if log_f and not log_f.closed:
        try:
            log_f.close()
        except Exception:
            pass  # Ignore errors during final logging attempt

    # Exit with the target process's exit code
    sys.exit(exit_code)

```

运行脚本

```shell
uv run llm_logger.py
```



在 Cline 设置中修改 API 提供者为 `Compatible`，使用本地 URL

![](image-20260312220416620.png)

然后提问 `hi` 就会在 `llm.log` 中看到“模型请求”和“模型返回”结果。



#### 协议内容

在模型请求中，有几个重要部分

- 工具：提供 Cline 内置的工具和外部的 MCP 服务工具
- 工具使用格式：使用 XML 风格的标签格式化工具信息，返回结果作为 XML 解析
	- `<tool_name>` 工具名
	- `<read_file>` 调用工具
	- `<execute_command>` 执行命令
	- `<use_mcp_tool>` 指定 MCP 服务的信息
	- `<attempt_completion>` 最终返回的结果
	- `<thinking>` 在使用工具前先使用 `<thinking>` 标签思考
	- `<task>` 要执行的任务
- 已连接的 MCP 服务器：提供可用的 MCP 服务器的详细信息
- `: OPENROUTER PROCESSING` 用于保持连接的注释，本身没有意义，防止等待时间过长断开连接
- `data: ...` 保存大模型的返回内容，每次在 `content` 中返回一小段信息

其基本思想是 ReAct

- Thought
- Action
- Observation

目的是让 AI 在不需要人为干预的情况下完成任务。



### RAG

RAG (Retrieval-Augmented Generation) 用于解决输入参考内容过大，导致模型难以从中找到问题相关信息的问题。RAG 能够先从参考内容中抽取相关内容，然后传给模型。



### Skills

```embed
title: "GitHub - libukai/awesome-agent-skills: Agent Skills 终极指南：快速入门、资源推荐、精选技能与工具组合 ｜The Ultimate Guide to Agent Skills: QuickStart, Resources, Features&Toolkit"
image: "https://opengraph.githubassets.com/d4a01f6fd03d4565898c59b63edae08e8127e6798afc917677ee487b8a922c1f/libukai/awesome-agent-skills"
description: "Agent Skills 终极指南：快速入门、资源推荐、精选技能与工具组合 ｜The Ultimate Guide to Agent Skills: QuickStart, Resources, Features&Toolkit - libukai/awesome-agent-skills"
url: "https://github.com/libukai/awesome-agent-skills"
```

```embed
title: "专为中国用户优化的Skills社区"
image: "https://cloudcache.tencent-cloud.com/qcloud/ui/static/open_source/fa130398-5145-4823-bcf7-b0d4d3127eba.png"
description: "专为中国用户优化的Skills社区，SkillHub 精选 Top 50 高质量 AI Skills，经过安全审核与多维度评估，助你发现最实用的 AI 技能。"
url: "https://skillhub.tencent.com/#featured"
```



### CLI

CLI 是最广泛使用的应用调用模式，例如 ffmpeg

```shell
ffmpeg -i xxx.mp3 xxx.mp4
```

实际上是程序自己通过 `--help` 给出用法，然后 AI 学习用法并通过规定好的格式操作程序。相较于 MCP，CLI 没有一套配置规范，完全依靠程序本身的命令行调用规范，并且它可以利用过去大量已经得到开发的程序 CLI，而且由于以前人类也是通过这种方式操作程序，因此当出现问题时，可以很容易手动调用来检查问题。



特别是 OpenCLI 是看起来比较实用的 AI 调用插件

```embed
title: "Release v1.6.8 · jackwener/opencli"
image: "https://opengraph.githubassets.com/c3c02805327fe2479d5abf17eb15bfc610f55ceb2a2418486a763d56acbfeb6e/jackwener/opencli/releases/tag/v1.6.8"
description: "What’s Changed refactor: centralize build path resolution by @jackwener in #807 fix: add safety boundaries to diagnostic output by @jackwener in #806 test: remove flaky bloomberg e2e tests by @jac…"
url: "https://github.com/jackwener/opencli/releases/tag/v1.6.8"
```



## 在线 Agent

### Manus

```embed
title: "Manus"
image: ""
description: "Manus 现已成为 Meta 的一部分一将 AI 带给全球的企业"
url: "https://manus.im/"
```



### 秒哒

百度开发的应用程序构建网站，输入详尽的需求文档，可以从该文档开始逐步构建项目代码

- 耗时较长，构建思路就是标准的 Agent
- 复杂需求不易实现，尝试制作魔塔游戏，其中地图显示存在问题，很难自动修正
- 默认提供的扫雷游戏比较简单，效果良好，说明适合制作简易程序



### FastGPT

可视化工作流、MCP 工具构建网站，可下载不同类型的工作流，也可以自定义

- 测试论文评价工作流，效果还可以
- 可以参考模板中使用的提示词格式
- 能够接入外部大模型 API



### 智谱清言

```embed
title: "智谱清言"
image: "https://chatglm.cn/favicon.ico"
description: "智谱清言是基于 GLM-5 的全能 AI 助手，支持精通对话、写作与编程。为你答疑解惑，激发创意，更能理解图片与文档，提升学习与工作效率。"
url: "https://chatglm.cn/"
```



## AI 笔记

### NotebookLM

顶级的 AI 笔记网站

- 可上传大量资料：pdf、视频、图片
- 基于资料提供 AI 问答，回答会标注来源
- 自动整理学习笔记
- 回答不自动保存
- 生成博客、ppt、讲解视频、思维导图、信息图



### ZeeLin

在 Web of Science 上选择文献，然后导出为 `Plain Text File` 格式，注意选择导出 `Abstract`，然后可以分析综述这些文件

- 无法直接获得原文件，因此只能基于摘要进行总结
- 似乎有部分内容是编造的，例如文章方法的不足之处，在摘要中并不存在



## AI 工具

### 咔片 ppt

上传文档自动制作 ppt，可选用多种模板

- 多模态能力不足，无法添加图片和公式
- 格式存在问题，例如参考文献引用未处理



### 笔格 ppt

上传文档自动制作 ppt，可选用多种模板，但是比较贵。仅仅测试提示词生成 ppt

- 模板很漂亮，但是缺乏与题目的配合，花里胡哨而无实际内容
- 格式标准，能够联网提供题目相关的资料并分类整合，但资料内容缺乏深度



### Aibiye

毕业论文、开题报告、文献综述、答辩 ppt 自动生成网站，价格昂贵。



### Q 文档

Word 文档批量生成网站，基于 Excel 表格结合 Word 文档模板，一键生成**法律文件、起诉书、律师函、合同、报告、成绩单、通知书、证书、邀请函、许可证等**各类文档。



## AI 绘图

### LibLibAI

AI 绘图网站，提供大量提示词模板，给定几张参考图就可用于生成高质量的图片

- 建议使用提示词模板而不是自然语言输入
- 可能出现生成失败的问题

![](image-20260310191601740.png|400)
