---
title: "MCP vs Skill：AI  Agent 扩展机制的对比与实战"
date: 2026-03-07
categories:
  - 技术
  - AI
tags:
  - MCP
  - Skill
  - AI Agent
  - OpenCode
---

本文对比介绍了 MCP（Model Context Protocol）和 Skill 两种 AI Agent 扩展机制，从概念、特点到编写配置，帮助你选择适合自己场景的方案。

## 一、MCP 与 Skill 的定位

在 AI Agent 生态中，如何让 AI 与外部世界连接并执行特定任务，是两个核心问题。MCP 和 Skill 分别从不同维度解决了这个问题：

### MCP：连接 AI 与外部系统的"插头"

MCP（Model Context Protocol）是一个**开放标准协议**，由 Anthropic 开源。它的设计理念类似于 USB-C 接口——为 AI 应用提供统一的连接方式。

```
┌──────────────────────────────────────────────┐
│  MCP (Model Context Protocol)                 │
│  ─────────────────────────────────────────    │
│  • 开放标准，跨平台支持                        │
│  • 统一 AI 与外部系统的通信方式                │
│  • 支持数据源、工具、工作流                    │
└──────────────────────────────────────────────┘

  [数据源] ←→ [工具] ←→ [工作流]
  (文件/数据库) (搜索引擎) (专用prompt)
```

### Skill：定义 AI 任务的"任务说明书"

Skill（技能）则是**任务级别的配置**，告诉 AI 应该按什么步骤执行什么任务。它更像是一份详细的工作流程文档。

```
┌──────────────────────────────────────────────┐
│  Skill (技能)                                 │
│  ─────────────────────────────────────────    │
│  • 任务流程配置                               │
│  • 包含步骤、依赖、定时任务                   │
│  • 通常是框架特定的实现                       │
└──────────────────────────────────────────────┘

   步骤1 → 步骤2 → 步骤3 → 发布
```

## 二、核心区别对比

| 维度 | MCP | Skill |
|------|-----|-------|
| **本质** | 通信协议 | 任务配置 |
| **层级** | 系统级 | 应用级 |
| **标准化** | 开放标准，跨平台 | 框架特定 |
| **作用** | 连接 AI 与外部系统 | 定义 AI 执行流程 |
| **开发方式** | 需实现 Server/Client | 编写 Markdown 配置 |
| **生态支持** | Claude、ChatGPT、VSCode、Cursor | 单一框架内 |

## 三、MCP 的特点与应用

### MCP 的核心优势

1. **标准化接口**：一次开发，多处使用
2. **跨平台兼容**：Claude、ChatGPT、Cursor、VSCode 都支持
3. **安全双向连接**：AI 系统与数据源安全通信
4. **丰富生态**：官方提供 Google Drive、Slack、GitHub 等预置 Server

### MCP 应用场景

- 连接企业数据库和 AI 助手
- 让 Claude Code 操作 Figma 设计
- 访问 Google Calendar、Notion 等个人数据
- 企业级 ChatBot 连接多数据源

### MCP 配置示例

**方式一：使用预置 Server（Claude Desktop）**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/files"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token"
      }
    }
  }
}
```

**方式二：自定义 Server（Python）**

```python
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("my-server")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="calculator",
            description="执行数学计算",
            inputSchema={
                "type": "object",
                "properties": {"expression": {"type": "string"}}
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "calculator":
        result = eval(arguments["expression"])
        return [TextContent(type="text", text=str(result))]
```

## 四、Skill 的特点与应用

### Skill 的核心优势

1. **配置简单**：Markdown 格式，易于编写
2. **任务聚焦**：针对特定场景预定义流程
3. **定时执行**：支持 Cron 定时任务
4. **依赖管理**：可声明前置要求和依赖包

### Skill 应用场景

- 自动化周报生成与发布
- 定时数据备份与同步
- 批量内容处理工作流
- 跨平台内容分发

### Skill 配置示例

**文件结构：**

```
~/.config/opencode/skills/my-skill/
├── SKILL.md          # 技能定义
├── requirements.txt  # Python 依赖
└── main.py          # 执行脚本
```

**SKILL.md 示例：**

```markdown
---
name: ai-news-weekly
description: 每周自动搜索 AI 资讯并发布
---

# 任务流程

## 前置要求
1. 部署 SearxNG 搜索服务
2. 完成平台认证

## 执行步骤
### 第一步：搜索资讯
使用关键词搜索 AI 最新动态...

### 第二步：生成内容
调用 LLM 整理结构化内容...

### 第三步：发布
发布到小红书、知乎、公众号

## 配置文件
```yaml
searxng:
  url: "http://localhost:8888"
cron:
  schedule: "0 8 * * 1"
```
```

## 五、如何选择？

根据实际需求选择合适的方案：

```
决策流程：

需要连接外部系统或服务？
│
├─ 是 ──→ 使用 MCP
│         • 数据库访问
│         • API 集成
│         • 文件系统操作
│         • 第三方服务
│
└─ 否 ──→ 使用 Skill
          • 自动化任务
          • 定时执行
          • 工作流编排
```

## 六、结合使用

实际上，MCP 和 Skill 可以**结合使用**：

- **MCP** 负责连接各种工具和数据源（搜索引擎、数据库、文件系统的连接器）
- **Skill** 负责编排任务流程（定义先做什么、后做什么、什么时候做）

这样既能享受 MCP 标准化连接的优势，又能通过 Skill 实现复杂的自动化任务。

## 总结

| 特性 | MCP | Skill |
|------|-----|-------|
| **定位** | 连接协议 | 任务配置 |
| **复杂度** | 较高（需编程） | 较低（配置为主） |
| **适用场景** | 系统集成 | 流程自动化 |
| **学习成本** | 较高 | 较低 |
| **扩展性** | 跨平台通用 | 框架限定 |

MCP 适合需要深度系统集成的场景，而 Skill 适合快速实现自动化任务。根据你的实际需求选择合适的方案，或者将两者结合使用，都能大幅提升 AI Agent 的能力。

---

如果你觉得本文有帮助，欢迎在评论区交流讨论！
