---
name: workflow-progress
description: 项目进度管理与任务追踪标准。支持本地 Markdown 模式 (默认) 和外部 MCP 工具模式 (Kanboard/Jira)。定义了状态流转、持久化配置存储及上下文恢复机制。
version: 1.0.0
tags: [project-management, kanban, mcp, documentation, progress-tracking]
---

# 进度管理与任务追踪工作流

本文档定义了 Vibe Coding 体系下的任务追踪标准。Agent 必须维护一个单一的事实来源 (Single Source of Truth, SSOT) 来反映项目的当前状态。

## 核心原则

- **实时性**：每完成一个具体的编码任务 (Task) 或修复 (Bugfix)，必须立即更新进度。
- **单一来源**：要么使用本地文件，要么使用外部系统。严禁同时维护两套状态，以免造成数据不同步。
- **持久化配置**：对于外部系统集成的配置信息，必须写入文件存储，以保证在 Agent 上下文清空后仍能恢复连接。

## 策略一：本地 Markdown 模式 (默认)

如果用户未指定外部工具，Agent 必须默认使用此模式。

### 文件规范

- **文件名**：`PROGRESS.md` (位于项目根目录)。
- **维护规则**：
  - Agent 必须在 `00_init` 阶段创建此文件。
  - 每次对话结束前，检查并更新此文件。

### 内容模板

```markdown
# Project Progress

## 📌 Status Overview
- **Phase**: [Design / Implementation / Testing / Release]
- **Current Sprint**: Sprint 1
- **Last Updated**: YYYY-MM-DD

## 🚀 Features (Backlog)
- [x] Feature A: User Login (Completed)
- [ ] Feature B: Payment Integration
    - [x] API Client implementation
    - [ ] Webhook handler
    - [ ] Frontend checkout page

## 🐛 Bug Fixes
- [ ] Fix: Login timeout on mobile (High Priority)

## 📝 Changelog / Notes
- 2024-03-20: Completed Feature A core logic.
```

## 策略二：外部 MCP 集成模式 (Kanboard/Jira)

当用户要求连接外部项目管理工具时，Agent 必须切换到此模式，并停止更新 PROGRESS.md (或仅保留指向面板的链接)。

### 1. 配置持久化 (Configuration Persistence)

为了防止上下文丢失，Agent 必须将外部工具的连接信息保存到项目的隐藏配置文件中。

- **文件路径**：`.vibe/config.toml`
- **写入时机**：用户首次提供工具信息 (如 Project ID, URL) 时。
- **读取时机**：Agent 启动新会话时，必须先读取此文件。

`.vibe/config.toml` 示例：

```toml
[project_management]
system = "kanboard"  # 或 "jira", "linear", "trello"
enabled = true

[project_management.kanboard]
project_id = "12"
board_url = "https://kanboard.example.com/board/12"
# 注意：敏感的 API Token 应通过环境变量获取，不应明文写入此文件
```

### 2. 标准看板流转 (Kanban Flow)

Agent 通过 MCP (Model Context Protocol) 工具操作外部系统时，必须遵循以下状态流转：

- **TODO -> Ready**：
  当 Analysis 阶段完成，AC (验收标准) 已确立。

- **Ready -> In Progress**：
  当 Agent 开始编写代码或测试时。
  - **动作**：将卡片移动到 "In Progress" 列。

- **In Progress -> Done**：
  当 TDD 循环完成，测试全绿，且代码已提交 (Git Commit)。
  - **动作**：将卡片移动到 "Done" 列。

### 3. 审计日志 (Audit Logging)

Agent 在外部系统中操作任务时，必须在任务/卡片的评论区 (Comment) 留下操作记录，以便追溯。

**记录内容**：
- 开始任务时的技术决策简述。
- 遇到的阻碍及解决方案。
- 相关的 Git Commit Hash。

**格式示例**：

> 🤖 Vibe Agent Update:
> 已完成核心逻辑实现。
> 状态: Passed all unit tests.
> Commit: feat: implement user service (sha: a1b2c3d)
> 下一步: 开始编写集成测试。

## Agent 交互协议

### 初始化检测

在每次会话开始时：
- Agent 检查 `.vibe/config.toml` 是否存在。
- 如果存在且 `project_management.enabled = true`：
  - 自动激活 MCP 工具 (如 `call_mcp_tool('kanboard', 'get_board_status')`)。
  - 告知用户："检测到关联的 Kanboard 项目，正在同步任务状态..."。
- 如果不存在：
  - 读取 `PROGRESS.md` 并向用户汇报当前进度。

### 切换指令

当用户说："把这个项目关联到我的 Kanboard，ID 是 5"：
1. 验证 Kanboard MCP 工具是否可用。
2. 创建/更新 `.vibe/config.toml`。
3. 询问："是否需要我根据当前的 PROGRESS.md 自动在 Kanboard 上创建任务？"
