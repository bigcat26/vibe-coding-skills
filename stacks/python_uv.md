---
name: stack-python-uv
description: Python 开发技术栈标准。强制使用 uv 进行包管理和环境隔离，集成 ruff (代码检查/格式化)、mypy (类型检查) 和 pytest (测试)。
version: 1.0.0
tags: [python, uv, ruff, mypy, pytest]
---

# Python 开发技术栈规范

本文档定义了 Vibe Coding 体系下 Python 项目的强制性技术栈和开发标准。核心理念是高性能、严格类型安全和环境可复现性。

## 工具链标准

Agent 必须使用以下工具，除非用户明确指定其他替代方案：

- **包管理器**: `uv` (替代 pip, poetry, pipenv, virtualenv)
- **代码检查与格式化**: `ruff` (替代 flake8, black, isort, pylint)
- **类型检查**: `mypy` (严格模式)
- **测试框架**: `pytest`
- **任务运行**: `uv run`

## 项目结构 (Layout)

必须使用 `src-layout` 结构，以避免导入路径错误并确保打包清洁。

- `project_name/`
  - `src/`
    - `project_name/`
      - `__init__.py`
      - `main.py`
  - `tests/`
    - `__init__.py`
    - `test_main.py`
  - `pyproject.toml` (配置中心)
  - `uv.lock` (锁定文件，严禁删除)
  - `.venv/` (由 uv 管理，忽略提交)
  - `.gitignore`
  - `README.md`

## 初始化工作流

当初始化一个新的 Python 项目时，按以下顺序执行：

- 创建目录结构。
- 初始化 uv 项目：
  - 命令：`uv init`
- 添加标准开发依赖：
  - 命令：`uv add --dev ruff mypy pytest pytest-cov`
- 创建 `.gitignore`：
  - 必须包含 `__pycache__/`, `*.py[cod]`, `.venv/`, `.coverage`, `htmlcov/`, `.mypy_cache/`, `.ruff_cache/`, `.pytest_cache/`。

## 配置标准 (pyproject.toml)

所有工具配置必须集中在 `pyproject.toml` 中。

### Ruff 配置
- 行长度：88 (保持 Black 兼容性)
- 目标版本：Python 3.10+
- 启用规则：
  - `E`: pycodestyle errors
  - `W`: pycodestyle warnings
  - `F`: Pyflakes
  - `I`: isort (导入排序)
  - `B`: flake8-bugbear
  - `UP`: pyupgrade (语法现代化)

### Mypy 配置
- 严格模式：启用 (`strict = true`)
- 忽略缺失导入：仅针对无类型提示的第三方库。

### Pytest 配置
- 测试路径：`testpaths = ["tests"]`
- 默认参数：`addopts = "-v --tb=short --strict-markers"`
- 覆盖率：通过 `pytest-cov` 插件，推荐 `addopts` 中追加 `--cov=src --cov-report=term-missing`。

## 开发指令集

Agent 必须使用以下命令执行开发任务，确保环境隔离：

- **添加依赖**：
  - `uv add <package_name>`
- **运行应用**：
  - `uv run python -m project_name.main`
- **运行测试**：
  - `uv run pytest`
- **格式化代码**：
  - `uv run ruff format .`
- **检查代码**：
  - `uv run ruff check . --fix`
- **类型检查**：
  - `uv run mypy src/`

## 编码规范

- **类型提示**：所有函数签名 (Signature) 必须包含参数和返回值的类型提示。
- **文档字符串**：公共模块、类和函数必须使用 Google 风格的 Docstrings。
- **导入规范**：优先使用绝对导入，避免相对导入。
- **路径处理**：必须使用 `pathlib.Path`，严禁使用 `os.path` 拼接字符串。
- **字符串格式化**：全部使用 f-strings (`f"{var}"`)。

## 错误处理

- 使用自定义异常类处理领域特定错误。
- 严禁使用裸露的 `except:` 语句，必须捕获具体异常。
- 在生产代码中使用 `logging` 模块，严禁使用 `print`。