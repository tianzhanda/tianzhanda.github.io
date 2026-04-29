---
title: "Hermes Web UI：给你的 AI Agent 装上图形化驾驶舱"
date: 2026-04-29
categories:
  - 技术
  - AI
tags:
  - Hermes
  - Web UI
  - AI Agent
  - 开源
  - 工具
---

> 一行命令，给你的 Hermes Agent 配上全功能 Web 管理面板。

## 什么是 Hermes Web UI

如果你在用 [Hermes Agent](https://github.com/NousResearch/hermes-agent)，那你一定习惯了 CLI 交互——在终端里打字、看日志、管理会话。但如果能有一个图形界面，可以像操作一个真正的 SaaS 产品一样管理你的 AI Agent 呢？

这就是 **Hermes Web UI** 做的事。它是一套自托管（self-hosted）的 Web 面板，让你通过浏览器完成一切：

- 与多模型 AI 聊天，支持流式输出
- 管理会话历史、定时任务、平台渠道
- 监控 Token 用量和费用
- 配置模型 Provider 和网关
- 甚至内置了一个网页版终端

项目地址：<https://github.com/EKKOLearnAI/hermes-web-ui>，MIT 开源。

### 一张图看懂架构

```
浏览器 → BFF (Koa, :8648) → Hermes 网关 (:8642)
                ↓
           Hermes CLI (会话、日志、版本)
                ↓
           ~/.hermes/config.yaml  (配置)
           ~/.hermes/auth.json    (凭证池)
```

前端是 Vue 3 + Naive UI，后端是 Koa 2 做 BFF 层，中间通过 REST API + SSE 实时通信。

---

## 核心功能一览

### 🤖 AI 聊天

- SSE 实时流式输出，非轮询，延迟低
- 多会话管理——创建、重命名、删除、切换
- **自建会话数据库**：本地 SQLite 存储，首次启动自动从 Hermes 同步历史会话
- 按来源分组（Telegram / Discord / Slack…），可折叠面板
- 活跃会话实时指示器——正在进行的会话置顶显示旋转图标
- Markdown 渲染 + 语法高亮 + 代码复制
- 工具调用详情展开（参数 / 结果）
- 文件上传与下载（支持 local / Docker / SSH 等多种后端）
- Ctrl+K 全局搜索所有对话
- 每个会话显示模型标签和上下文 Token 用量

### 🔌 平台渠道管理

一个页面统一配置 **8 个平台**：

| 平台 | 功能亮点 |
|------|----------|
| Telegram | Bot Token、提及控制、表情回应、自由回复 |
| Discord | Bot Token、自动线程、频道白/黑名单 |
| Slack | Bot Token、提及控制 |
| WhatsApp | 启用/禁用、提及模式 |
| Matrix | Access Token、Homeserver、自动线程 |
| 飞书 | App ID / Secret、提及控制 |
| 微信 | **扫码登录**（浏览器扫码，自动保存凭证） |
| 企业微信 | Bot ID / Secret |

配置变更后自动重启网关，无需手动操作。

### 📊 用量分析

- Token 总用量统计（输入 / 输出）
- 会话数及日均统计
- 预估费用追踪 + 缓存命中率
- 模型使用分布图
- 30 天每日趋势（柱状图 + 数据表格）

### ⏰ 定时任务

- 创建、编辑、暂停、恢复、删除 Cron 任务
- 立即触发执行
- Cron 表达式快捷预设

### 🧠 模型管理

- 从凭证池自动发现模型（`~/.hermes/auth.json`）
- 从每个 Provider 端点获取可用模型列表
- 添加、更新、删除 Provider（支持自定义 OpenAI 兼容端点）
- Provider URL 自动检测，支持非标准 API 版本（如 `/v4`）
- OpenAI Codex 和 Nous Portal OAuth 登录

### 👥 群聊功能

- 多 Agent 聊天房间，通过 Socket.IO 实时通信
- **@提及路由**——提及某 Agent 触发上下文回复
- 上下文压缩——历史消息超 Token 阈值时自动摘要
- 房间创建、删除和邀请码管理
- Agent 独立 Profile 配置
- 移动端响应式布局

### 📂 更多功能

| 功能 | 说明 |
|------|------|
| **多配置文件** | 创建、克隆、导入/导出 Profile，多网关管理 |
| **技能与记忆** | 浏览已安装技能、查看详情、管理记忆 |
| **文件浏览器** | 浏览远程后端文件，支持上传下载 |
| **日志查看** | Agent / Gateway / Error 日志，按级别和关键词过滤 |
| **Web 终端** | 内置 node-pty + xterm 终端，直接在浏览器里操作 |
| **认证** | Token 认证 + 可选用户名密码、支持禁用认证 |

---

## 快速安装

### npm 全局安装（推荐）

```bash
npm install -g hermes-web-ui
hermes-web-ui start
```

打开 **<http://localhost:8648>** 即可使用。

### 一键安装脚本

自动检测系统并安装 Node.js（如未安装）：

```bash
bash <(curl -fsSL https://cdn.jsdelivr.net/gh/EKKOLearnAI/hermes-web-ui@main/scripts/setup.sh)
```

### Docker Compose

```bash
# 使用预构建镜像
WEBUI_IMAGE=ekkoye8888/hermes-web-ui:latest \
  docker compose up -d hermes-agent hermes-webui

# 或从源码构建
docker compose up -d --build hermes-agent hermes-webui
```

打开 **<http://localhost:6060>**

### CLI 命令

| 命令 | 说明 |
|------|------|
| `hermes-web-ui start` | 后台守护进程启动 |
| `hermes-web-ui start --port 9000` | 自定义端口 |
| `hermes-web-ui stop` | 停止 |
| `hermes-web-ui restart` | 重启 |
| `hermes-web-ui status` | 查看状态 |
| `hermes-web-ui update` | 更新到最新版 |
| `hermes-web-ui -v` | 查看版本 |

### 自动配置

启动时 BFF 服务器会自动：

- 校验并补全 `~/.hermes/config.yaml` 配置
- 修改时备份原配置到 `.bak`
- 检测并启动网关（如未运行）
- 解决端口冲突
- 启动成功后自动打开浏览器

---

## 技术栈

| 层 | 技术 |
|:---|:---|
| **前端** | Vue 3 + TypeScript + Vite + Naive UI + Pinia + vue-i18n |
| **后端** | Koa 2（BFF 服务器）+ node-pty（Web 终端） |
| **实时通信** | SSE（聊天流式）+ Socket.IO（群聊） |
| **构建** | Vite + esbuild |

项目设计为**多 Agent 可扩展架构**——所有 Hermes 相关代码按命名空间组织在 `hermes/` 目录下，后续接入其他 Agent 也不会冲突。

---

## 总结

Hermes Web UI 把 Hermes Agent 从纯 CLI 工具升级成了完整的可视化平台。如果你已经在用 Hermes，装一个 Web UI 几乎零成本——npm 一行命令装好，立刻获得：

- 更直观的聊天体验（多会话、流式输出、历史记录）
- 一站式的平台配置管理
- 可视化的用量监控和费用追踪
- 浏览器里的终端

安装仅需一行命令，推荐所有 Hermes 用户试试。

---

*项目版本：v0.5.0 | 许可证：MIT*
