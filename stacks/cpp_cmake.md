---
name: stack-cpp-cmake
description: C++ 开发技术栈标准。使用 CMake + Conan + Ninja 构建工作流，集成 Clang-Tidy (静态分析)、Clang-Format (格式化) 和 GTest (测试)。
version: 1.0.0
tags: [cpp, cmake, conan, ninja, gtest, clang-tidy]
---

# C++ 开发技术栈规范

本文档定义了 Vibe Coding 体系下 C++ 项目的强制性技术栈和开发标准。核心理念是现代 C++ (C++17/20)、可复现构建和严格的静态分析。

## 工具链标准

Agent 必须使用以下工具，除非用户明确指定其他替代方案：

- **构建系统**: `CMake` (>= 3.15) + `Ninja`
- **包管理器**: `Conan` (>= 2.0)
- **静态分析**: `Clang-Tidy`
- **代码格式化**: `Clang-Format`
- **测试框架**: `Google Test` (GTest)
- **编译器**: 推荐 `Clang` 或 `GCC` (支持 C++17/20)

## 项目结构 (Layout)

- `project_name/`
  - `src/`
    - `project_name/`
      - `main.cpp`
      - `*.cpp` / `*.h`
  - `include/`
    - `project_name/`
      - `*.h` / `*.hpp` (公共头文件)
  - `tests/`
    - `test_main.cpp`
  - `CMakeLists.txt` (顶层构建配置)
  - `CMakePresets.json` (构建预设)
  - `conanfile.txt` 或 `conanfile.py` (依赖描述)
  - `.clang-tidy` (静态分析配置)
  - `.clang-format` (格式化配置)
  - `.gitignore`
  - `README.md`

## 初始化工作流

当初始化一个新的 C++ 项目时，按以下顺序执行：

- 创建目录结构 (`src/`, `include/`, `tests/`)。
- 创建 `CMakeLists.txt`：
  - 设置最低 CMake 版本、项目名称、C++ 标准 (17 或 20)。
  - 配置 `src/` 和 `include/` 目录。
  - 添加 GTest 测试目标。
- 创建 `CMakePresets.json`：
  - 定义 `debug` 和 `release` 预设，使用 Ninja 生成器。
- 创建 `conanfile.txt`：
  - 声明项目依赖 (如 `gtest/1.14.0`)。
- 创建 `.gitignore`：
  - 必须包含 `.build/`, `build/`, `CMakeUserPresets.json`, `*.o`, `*.a`, `*.so`, `*.dylib`。

## 构建工作流

### 确定构建目录

如果用户指定了构建类型 (如 `debug` 或 `release`)，查找 `CMakePresets.json` 中对应预设的 `binaryDir` 配置。否则默认使用 `.build` 目录。

### 标准构建流程

```bash
# 1. 创建构建目录
mkdir -p <build_directory>

# 2. 安装 Conan 依赖
conan install . --output-folder=<build_directory> --build=missing --settings=build_type=<BUILD_TYPE> -r conancenter

# 3. 配置 CMake
cmake --preset <preset_name>

# 4. 构建
cmake --build --preset <preset_name>
```

### Conan 参数说明

- `--output-folder`：指定依赖输出目录
- `--build=missing`：本地构建缺失的包
- `--settings=build_type=<BUILD_TYPE>`：指定构建类型 (Debug/Release)
- `-r conancenter`：使用 conancenter 仓库

## 开发指令集

Agent 必须使用以下命令执行开发任务：

- **安装依赖**：
  - `conan install . --output-folder=.build --build=missing`
- **配置项目**：
  - `cmake --preset <preset_name>`
- **构建项目**：
  - `cmake --build --preset <preset_name>`
- **运行测试**：
  - `ctest --preset <preset_name>` 或 `cmake --build --preset <preset_name> --target test`
- **格式化代码**：
  - `find src include tests -name '*.cpp' -o -name '*.h' | xargs clang-format -i`
- **静态分析**：
  - `clang-tidy src/**/*.cpp -- -I include/`

## 编码规范

- **C++ 标准**：优先使用 C++17，新项目推荐 C++20。
- **头文件保护**：使用 `#pragma once`，不使用传统的 include guard。
- **命名规范**：
  - 类名：`PascalCase` (如 `UserService`)
  - 函数/方法：`snake_case` (如 `get_user`)
  - 常量/枚举值：`kPascalCase` (如 `kMaxRetries`)
  - 成员变量：`snake_case_` (尾部下划线，如 `name_`)
- **智能指针**：严禁使用裸指针 (`new`/`delete`)，必须使用 `std::unique_ptr` 或 `std::shared_ptr`。
- **字符串**：优先使用 `std::string_view` 作为函数参数，`std::string` 作为存储。

## 错误处理

- 优先使用异常 (Exceptions) 处理可恢复错误，使用 `std::expected` (C++23) 或返回值处理预期内的失败。
- 利用 RAII 确保资源安全释放，严禁手动 `new`/`delete`。
- 严禁捕获所有异常后静默忽略 (`catch(...) {}`)。
- 在日志中使用 `spdlog` 或类似结构化日志库，严禁使用 `std::cout` 输出调试信息到生产代码。

## 故障排查

- **Conan 包未找到**：检查 conancenter 仓库是否正确配置。
- **CMake 配置失败**：检查 `CMakePresets.json` 中的预设配置。
- **首次构建耗时长**：Conan 会自动下载并构建缺失的包，属正常现象。
