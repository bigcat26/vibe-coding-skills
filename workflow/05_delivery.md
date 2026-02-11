---
name: workflow-delivery
description: 持续集成与交付 (CI/CD) 标准。涵盖 Docker 容器化规范及主流 CI 平台 (GitHub Actions, GitLab CI, Jenkins) 的流水线配置生成。
version: 1.0.0
tags: [ci-cd, docker, delivery, github-actions, gitlab-ci, jenkins]
---

# 交付与构建工作流

本文档定义了 Vibe Coding 体系下的软件交付标准。Agent 必须确保代码不仅能在本地运行，还能在标准化环境中自动构建和测试。

## 核心原则

- **基础设施即代码 (IaC)**：所有构建和部署逻辑必须定义在代码仓库的配置文件中。
- **一次构建，随处运行**：优先使用 Docker 容器化交付。
- **流水线即文档**：CI/CD 配置文件应清晰描述构建、测试和发布的步骤。

## Docker 容器化规范

如果项目需要交付容器镜像，Agent 必须提供 `Dockerfile`。

### Dockerfile 最佳实践
- **多阶段构建 (Multi-stage Build)**：分离构建环境和运行环境，减小镜像体积。
- **基础镜像**：使用轻量级镜像 (如 `python:3.11-slim`, `node:20-alpine`, `gcc:13-bookworm` 等)。
- **层缓存优化**：先拷贝依赖描述文件 (如 `pyproject.toml`, `package.json`, `conanfile.txt`) 并安装依赖，再拷贝源代码。
- **非 Root 用户**：在运行阶段必须创建一个非特权用户运行应用。

## CI/CD 平台选择协议

在生成 CI 配置文件之前，Agent 必须通过以下逻辑确定目标平台：

1.  **检测**：检查项目根目录是否存在特定文件夹 (如 `.github`, `.gitlab-ci.yml`, `Jenkinsfile`)。
2.  **询问**：如果未检测到，必须询问用户：“请选择目标 CI/CD 平台：GitHub Actions, GitLab CI, 还是 Jenkins？”

### 选项一：GitHub Actions (默认推荐)

- **文件路径**：`.github/workflows/ci.yml`
- **触发条件**：
  - Push 到 `main` / `develop` 分支。
  - 针对 `main` / `develop` 的 Pull Request。
- **标准作业 (Jobs)**：
  - **Test**: 检出代码 -> 安装工具链 -> 安装依赖 -> 运行 Lint -> 运行 Test。
  - **Build**: (仅在 Tag 推送时) 构建 Docker 镜像并推送至 GHCR 或 Docker Hub。
  - *(Python 示例: 安装 uv -> `uv sync` -> `ruff check` -> `pytest`)*

### 选项二：GitLab CI

- **文件路径**：`.gitlab-ci.yml`
- **阶段 (Stages)**：`lint`, `test`, `build`, `deploy`。
- **缓存策略**：缓存 `.venv` 或 `uv` 缓存目录以加速构建。
- **Docker-in-Docker**：如果需要构建镜像，正确配置 `dind` 服务。

### 选项三：Jenkins

- **文件路径**：`Jenkinsfile`
- **语法**：必须使用 **Declarative Pipeline** (声明式流水线)。
- **代理 (Agent)**：推荐使用 Docker 容器化代理以保证环境隔离。
- **步骤**：
  - `stage('Install')`: 安装依赖
  - `stage('Test')`: 运行测试
  - `stage('Lint')`: 运行静态检查
  - *(Python 示例: `agent { docker { image 'python:3.11' } }`, `sh 'uv sync'`, `sh 'uv run pytest'`)*

## 生成产物规范

Agent 生成的 CI 配置文件必须满足：

- **环境一致性**：CI 环境中的语言/运行时版本必须与项目配置文件中声明的一致。
- **依赖锁**：必须使用锁定文件安装依赖 (如 Python 的 `uv sync --frozen`、Node 的 `npm ci`)，严禁在 CI 中升级依赖。
- **故障反馈**：配置测试报告输出 (如 JUnit XML)，以便 CI 平台展示测试结果。

## Agent 交互示例

**场景**：用户完成了一个 Python 项目开发，要求配置自动测试。

**Agent 响应**：
> "已检测到这是一个 Python 项目。为了配置自动测试流水线，请问您使用的 CI 平台是：
> 1. **GitHub Actions** (推荐，适合开源或 GitHub 托管项目)
> 2. **GitLab CI** (适合企业私有仓库)
> 3. **Jenkins** (适合传统构建服务器)
>
> 请选择一个，我将为您生成对应的配置文件。"