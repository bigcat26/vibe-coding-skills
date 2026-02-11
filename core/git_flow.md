---
name: core-git-flow
description: 完整的 Git 工作流规范。涵盖 Git Flow 分支模型 (Feature/Release/Hotfix)、语义化版本控制 (SemVer)、Conventional Commits 提交标准以及 Agent 自动化交互协议。
version: 1.2.0
tags: [git-flow, semver, conventional-commits, branching-strategy]
---

# Git 工作流与版本控制规范

本文档定义了 Vibe Coding 体系下的标准开发流。Agent 必须严格遵守 Git Flow 分支模型、语义化版本控制以及 Conventional Commits 提交规范。

## Git Flow 分支模型

项目必须维护两类长期分支和三类短期分支。

### 长期分支 (Long-lived Branches)

- **main (或 master)**
  - 定义：生产环境代码库。
  - 规则：永远处于可部署状态。严禁直接在 main 分支上进行 Commit，只能通过 Pull Request 或 Merge 合并代码。
  - 标签：所有生产发布的版本号 (Tag) 必须打在此分支上。

- **develop**
  - 定义：集成开发分支。
  - 规则：包含下一次发布的所有已完成功能。这是所有 Feature 分支的基准点。

### 短期分支 (Supporting Branches)

- **Feature 分支**
  - 命名：`feature/功能名` (例如 `feature/user-auth`)
  - 来源：`develop`
  - 去向：`develop`
  - 用途：开发新功能。

- **Release 分支**
  - 命名：`release/x.y.z` (例如 `release/1.2.0`)
  - 来源：`develop`
  - 去向：`main` 和 `develop`
  - 用途：准备新版本的发布。只允许进行文档更新、Bug 修复和版本号修改，严禁添加新功能。

- **Hotfix 分支**
  - 命名：`hotfix/问题描述` (例如 `hotfix/security-patch`)
  - 来源：`main`
  - 去向：`main` 和 `develop`
  - 用途：修复生产环境的紧急 Bug。这是唯一允许直接从 `main` 切出的分支类型。

## 语义化版本控制 (Semantic Versioning)

版本号格式必须遵循 `MAJOR.MINOR.PATCH` (主版本号.次版本号.修订号)。

- **MAJOR (主版本号)**
  - 当做了不兼容的 API 修改时增加。
  - 例如：`1.0.0` -> `2.0.0`

- **MINOR (次版本号)**
  - 当做了向下兼容的功能性新增时增加。
  - 例如：`1.1.0` -> `1.2.0`

- **PATCH (修订号)**
  - 当做了向下兼容的问题修正时增加。
  - 例如：`1.1.1` -> `1.1.2`

## 提交信息规范 (Conventional Commits)

所有提交信息必须符合以下格式：

`类型(范围): 描述`

### Type (类型)

- **feat**: 新功能 (Feature)。
- **fix**: 修复 Bug。
- **docs**: 仅文档更改。
- **style**: 代码格式修改 (不影响逻辑)。
- **refactor**: 代码重构 (无新功能或 Bug 修复)。
- **perf**: 性能优化。
- **test**: 添加或修改测试。
- **chore**: 构建过程或辅助工具变动。
- **build**: 影响构建系统或外部依赖的更改。
- **ci**: CI 配置文件更改。

### Subject (标题)

- 使用祈使句，一般现在时 (如 "change" 而不是 "changed")。
- 首字母小写。
- 结尾不加句号。
- 不超过 50 个字符。

### Footer (页脚 - 可选)

- **BREAKING CHANGE**: 如果包含破坏性变更，必须以此开头。
- **Closes**: 关联 Issue (例如 `Closes #123`)。

## Agent 自动化交互协议

Agent 在执行 Git 操作时，必须根据当前上下文自动判断并执行以下流程。

### 场景一：新功能开发 (Feature Development)

- **触发条件**：用户要求开发新功能。
- **Agent 行为**：
  - 检查当前是否在 `develop` 分支。
  - 如果不是，提示用户并请求切换。
  - 如果是，自动创建并切换到 `feature/xxx` 分支。
  - 开发过程中遵循 TDD 和原子提交。

### 场景二：紧急修复 (Hotfix)

- **触发条件**：用户报告生产环境 Bug 或要求 Hotfix。
- **Agent 行为**：
  - 检查当前是否在 `main` 或 `master` 分支。
  - 自动创建并切换到 `hotfix/xxx` 分支。
  - 修复完成后，提示用户该分支需要同时合并回 `main` 和 `develop`。

### 场景三：版本发布 (Release)

- **触发条件**：用户要求准备发布新版本。
- **Agent 行为**：
  - 基于 `develop` 创建 `release/x.y.z` 分支。
  - 协助用户更新项目配置文件 (如 `pyproject.toml`, `package.json`) 中的版本号。
  - 生成 CHANGELOG。
  - 提示用户进行最终测试。

### 场景四：常规开发与非标准状态 (Manual Confirmation)

- **触发条件**：
  - 用户未指定分支策略。
  - 用户仅要求简单修改。
  - 当前处于非 `develop` 且非 `main` 的游离状态。
- **Agent 行为**：
  - 严禁自动执行 `git commit`。
  - Agent 完成代码修改后，必须输出 Git Commit 提案：
    - 变更文件清单 (`git status` 结果)。
    - 建议的 Commit Message (符合 Conventional Commits)。
    - 待执行命令 (`git add . && git commit -m "..."`)。
  - 等待用户确认或手动执行。