---
title: "Hermes Agent 完全指南：一款会自我进化的开源 AI Agent"
date: 2026-04-29
categories:
  - 技术
  - AI
tags:
  - AI Agent
  - Hermes
  - 开源
  - 工具
---

> 一行命令安装，5 分钟上手，它会自己变得越来越聪明。

## 一、Hermes Agent 是什么

Hermes Agent 是由 **Nous Research**（Hermes、Nomor 等开源模型的作者）开发的**自主学习型 AI Agent**，于 2026 年 2 月开源发布。它不是简单的聊天机器人或 IDE 插件，而是一个能够持续学习、不断进化的 autonomous agent。

### 核心特点

| 特性 | 说明 |
|:---|:---|
| **自我进化** | 唯一内置学习循环的 Agent——创建技能、改善技能、跨会话记忆 |
| **持久记忆** | SQLite + FTS5 全文本搜索，跨会话积累上下文和用户模型 |
| **技能系统** | 从经验中自动生成技能（skill），可在项目中复用 |
| **多模型支持** | OpenRouter（200+ 模型）、OpenAI、Anthropic、Ollama 等 |
| **多平台接入** | CLI、Telegram、Discord、Slack、WhatsApp 等 15+ 平台 |
| **6 种后端** | 本地、Docker、SSH、Daytona、Singularity、Modal |
| **47+ 内置工具** | 网页搜索、文件编辑、终端执行、浏览器自动化、MCP 等 |
| **免费开源** | MIT/Apache 2.0 许可，只需支付 LLM API 调用费用 |

### 与同类产品对比

| | Hermes Agent | Claude Code | Cursor |
|:|:|:|:|
| 费用 | 免费（+API 费用） | ~$18/月 | $20/月（Pro） |
| 内置工具 | 47+ | ~15 | ~20 |
| 记忆系统 | 持久跨会话 | 无 | 无 |
| 自定义技能 | 支持自动创建 | 不支持 | 不支持 |
| 开源 | 是 | 否 | 否 |

---

## 二、安装

### 快速一键安装（Linux / macOS / WSL2）

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

安装脚本自动完成以下所有步骤：

- 检测并安装 `uv`（Python 包管理器）
- 检测并安装 Python 3.11、Node.js v22、ripgrep、ffmpeg
- 克隆代码到本地目录
- 配置 PATH 和虚拟环境
- 设置全局 `hermes` 命令

> **注意**：原生 Windows 不支持，需通过 WSL2（Windows Subsystem for Linux）。

### 完成后重载 Shell

```bash
source ~/.bashrc   # bash 用户
source ~/.zshrc    # zsh 用户
```

### 验证安装

```bash
hermes --version
# 输出示例: hermes-agent v0.8.0
```

### 使用 Nix（可选）

```bash
nix run github:NousResearch/hermes-agent
```

---

## 三、配置与首次启动

### 首次运行配置向导

```bash
hermes setup
```

配置向导会引导你完成以下步骤：

1. **选择 LLM 提供商** - OpenRouter / OpenAI / Anthropic / Ollama 等
2. **输入 API Key** - 安全输入你的 API 密钥
3. **配置工具集** - 选择需要启用的工具
4. **设置基本偏好** - 语言、交互风格等

### 交互式使用

```bash
hermes              # 进入交互式 CLI 对话
```

进入后即可像与真人助手对话一样使用 Hermes。

### 常用命令一览

| 命令 | 说明 |
|:---|:---|
| `hermes` | 启动交互式 CLI 对话 |
| `hermes setup` | 运行完整配置向导 |
| `hermes model` | 切换 LLM 模型 |
| `hermes tools` | 配置启用的工具集 |
| `hermes config` | 查看/编辑配置 |
| `hermes doctor` | 诊断问题 |
| `hermes update` | 更新到最新版本 |
| `hermes gateway` | 启动消息网关 |
| `hermes claw migrate` | 从 OpenClaw 迁移配置 |

---

## 四、作为 Python 库使用

Hermes Agent 也可作为 Python 包导入你的项目中使用。

### 安装为 Python 库

```bash
# 使用 pip
pip install git+https://github.com/NousResearch/hermes-agent.git

# 或使用 uv
uv pip install git+https://github.com/NousResearch/hermes-agent.git
```

也可在 `requirements.txt` 中添加：

```
hermes-agent @ git+https://github.com/NousResearch/hermes-agent.git
```

### 基础用法

```python
from hermes_agent import AIAgent

agent = AIAgent(
    model="anthropic/claude-opus-4.6",
    api_key="your-api-key"  # 或设置 OPENROUTER_API_KEY 环境变量
)

response = agent.chat("帮我写一个快速排序函数")
print(response)
```

### 关键构造参数

| 参数 | 类型 | 默认值 | 说明 |
|:---|:---|:---|:---|
| `model` | str | `"anthropic/claude-opus-4.6"` | 模型名称（OpenRouter 格式） |
| `quiet_mode` | bool | `False` | 静默模式，抑制 CLI 输出 |
| `enabled_toolsets` | List[str] | `None` | 启用的工具集白名单 |
| `max_iterations` | int | `90` | 单次对话最大迭代次数 |
| `skip_context_files` | bool | `False` | 跳过加载 AGENTS.md 文件 |
| `api_key` | str | `None` | API Key（会 fallback 到环境变量） |
| `save_trajectories` | bool | `False` | 将对话保存为 JSONL 格式 |

---

## 五、高级功能详解

### 1. 消息网关（多平台接入）

```bash
hermes gateway              # 启动网关
hermes gateway enable-autostart  # 设置开机自启
hermes gateway status     # 查看网关状态
```

支持的平台包括 Telegram、Discord、Slack、WhatsApp、Signal、Matrix、飞书、钉钉、企业微信等 15+ 平台。

### 2. 定时任务（Cron）

```bash
hermes cron          # 创建定时任务
```

支持自然语言描述或标准 Cron 表达式，可定时执行任务并通过任意平台推送结果。

### 3. MCP 集成

```bash
hermes mcp          # 配置 MCP 服务器
```

可连接 Model Context Protocol 服务器，扩展更多工具能力。

### 4. 技能系统

```bash
hermes skill create  # 从任务创建新技能
hermes skill list   # 列出已有技能
```

Hermes 可从复杂任务执行中自动提取和创建可复用技能。

### 5. 子 Agent 并行

Agent 可生成隔离的子 Agent（Subagent）并行处理多任务，最多支持 3 个并发子进程。

### 6. 代码执行

Agent 可编写 Python 脚本，通过 Unix Domain Socket RPC 调用 Hermes 工具，适合复杂的多步骤工具调用流水线。

### 7. 浏览器自动化

内置 10+ 浏览器自动化工具，可进行网页抓取、表单填写等操作。

### 8. IDE 集成（ACP）

通过 ACP（Agent Communication Protocol）可集成到 VS Code、Zed、JetBrains 等编辑器。

### 9. 批量处理

支持批量轨迹生成，可用于训练数据生成或评估。

---

## 六、配置说明

### 配置文件位置

配置文件存储在 `~/.hermes/config.yaml`，支持 YAML 格式配置。

### 主要配置项示例

```yaml
model: "anthropic/claude-opus-4.6"
api_key: "${OPENROUTER_API_KEY}"
quiet_mode: false

code_execution:
  mode: "project"

providers:
  openrouter:
    api_key: "${OPENROUTER_API_KEY}"
  anthropic:
    api_key: "${ANTHROPIC_API_KEY}"
  openai:
    api_key: "${OPENAI_API_KEY}"
```

### 环境变量方式

推荐使用环境变量存储敏感信息：

```bash
export OPENROUTER_API_KEY="***"
export OPENAI_API_KEY="***"
export ANTHROPIC_API_KEY="***"
```

---

## 七、VPS 部署

### 最低配置要求

- $5/月 VPS（DigitalOcean Droplet、Hetzner CX22、Vultr 等）
- Ubuntu 22.04 LTS
- 512MB 内存 + 10GB 存储

### 部署步骤

```bash
# 1. 更新系统
apt update && apt upgrade -y

# 2. 安装 Git（唯一前提）
apt install git -y

# 3. 运行安装脚本
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 4. 引导配置
hermes setup

# 5. 启用网关自启
hermes gateway enable-autostart

# 6. 检查状态
hermes gateway status
```

---

## 八、故障排查

| 问题 | 解决方案 |
|:---|:---|
| `hermes: command not found` | 执行 `source ~/.bashrc` 重载 PATH |
| API key 未设置 | 运行 `hermes model` 或 `hermes config set OPENROUTER_API_KEY xxx` |
| 安装失败 | 确认已安装 `git`，重试安装脚本 |
| 配置丢失 | 依次执行 `hermes config check` → `hermes config migrate` |
| 其他问题 | 运行 `hermes doctor` 自动诊断 |

---

## 九、常用工具集

| 工具集 | 包含工具 |
|:---|:---|
| `default` | 基础工具（文件读写、搜索、执行终端命令） |
| `web` | 网页搜索、提取 |
| `browser` | 浏览器自动化（10+ 工具） |
| `memory` | 记忆系统工具 |
| `code_execution` | Python 脚本执行 |
| `delegate` | 子 Agent 委托 |
| `mcp` | MCP 客户端 |

---

## 十、资源链接

| 资源 | 链接 |
|:---|:---|
| GitHub 仓库 | https://github.com/NousResearch/hermes-agent |
| 官方文档 | https://hermes-agent.nousresearch.com/docs |
| 官方博客 | https://hermes-agent.ai/blog |
| 技能市场 | https://agentskills.io |
| 在线演示 | https://hermes-agent.ai |

---

## 总结

Hermes Agent 是一款具有**自我进化能力**的革命性开源 AI Agent。其核心优势在于：

1. **持久记忆** - 不像传统工具那样"每次都是从零开始"
2. **自我进化** - 能从经验中创建和改进技能
3. **免费开源** - MIT/Apache 2.0 许可，无锁定
4. **灵活部署** - 从 5 美元 VPS 到 serverless 都可以
5. **多平台接入** - 不仅可以在终端使用

安装仅需一行命令，5 分钟内即可开始使用。推荐对 AI Agent 有兴趣的开发者尝试。

---

*文档最后更新：2026年4月*
