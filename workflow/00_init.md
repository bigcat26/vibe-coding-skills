---
name: workflow-init
description: 项目初始化标准流程。涵盖目录创建、Git 仓库初始化、依赖管理初始化 (.gitignore) 及首次提交规范。
version: 1.0.0
tags: [workflow, init, scaffolding]
---

# 项目初始化工作流

本文档定义了 Vibe Coding 体系下的项目启动标准。无论使用何种编程语言，Agent 必须执行以下通用步骤，并调用特定技术栈 (Stack) 的指令。

## 初始化前置检查

在执行任何写操作前，Agent 必须：

- 确认目标目录为空，或仅包含非冲突文件。
- 确认已获取项目名称和简要描述。
- 确认已识别所需的技术栈 (如 Python, C++, TypeScript)。

## 标准执行步骤

### 步骤一：基础骨架搭建

- 创建项目根目录。
- 创建 `README.md`，内容必须包含：
  - 项目名称 (一级标题)。
  - 项目描述。
  - **Prerequisites** (依赖环境，如 `uv`, `node`)。
  - **Usage** (启动指令)。

### 步骤二：Git 仓库初始化

- 执行 `git init`。
- 创建 `.gitignore` 文件。
  - 必须根据识别到的技术栈，写入通用的忽略规则 (如 Python 的 `__pycache__`, `.venv`; Node 的 `node_modules`; 系统的 `.DS_Store`)。
- 创建并切换到 `develop` 分支。
  - `git checkout -b develop`
  - 根据 Git Flow 规范，`develop` 是主要的开发分支，初始代码不应直接停留在 master/main。

### 步骤三：技术栈初始化 (Stack Injection)

- **读取对应的 Stack 文档** (如 `stacks/python_uv.md`)。
- 执行文档中定义的初始化命令 (如 `uv init` 或 `npm init`)。
- 执行文档中定义的开发依赖安装命令 (如 `uv add --dev ruff pytest`)。
- 调整目录结构以符合 Stack 文档要求的 Layout (如 `src/` 结构)。

### 步骤四：首次提交

- 验证所有生成的文件已加入暂存区。
- 执行原子提交。
- 命令：`git add . && git commit -m "chore: initial commit"`

## Agent 交互协议

- **自动化决策**：不要询问“你要把文件放在哪？”或“你要用什么名字的文件夹？”，默认使用项目名称作为目录名或在当前目录下操作。
- **安全警告**：如果当前目录不为空且存在覆盖风险，Agent **必须**暂停并请求用户确认。
- **环境一致性**：在初始化 `pyproject.toml` 或 `package.json` 时，必须锁定版本号或使用宽泛但安全的版本约束。