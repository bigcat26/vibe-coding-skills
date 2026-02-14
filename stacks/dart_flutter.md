---
name: stack-dart-flutter
description: Dart/Flutter App 开发技术栈标准。使用 Flutter 框架进行跨平台移动应用开发，集成 Riverpod (状态管理)、GoRouter (路由)、flutter_test (测试) 和 dart_code_metrics (静态分析)。
version: 1.0.0
tags: [dart, flutter, riverpod, go_router, freezed]
---

# Dart / Flutter App 开发技术栈规范

本文档定义了 Vibe Coding 体系下 Dart/Flutter 移动应用项目的强制性技术栈和开发标准。核心理念是声明式 UI、不可变状态管理和严格的分层架构。

## 工具链标准

Agent 必须使用以下工具，除非用户明确指定其他替代方案：

- **SDK**: `Flutter` (Stable Channel, >= 3.19)
- **语言**: `Dart` (>= 3.3，启用 null safety)
- **状态管理**: `Riverpod` (flutter_riverpod + riverpod_annotation + riverpod_generator)
- **路由**: `GoRouter` (go_router)
- **网络请求**: `Dio` + `Retrofit`
- **数据模型**: `Freezed` + `json_serializable` (不可变模型 + JSON 序列化)
- **本地存储**: `Hive` 或 `SharedPreferences`
- **代码生成**: `build_runner`
- **代码检查**: `flutter_lints` (官方推荐规则集) + `custom_lint_rules`
- **代码格式化**: `dart format` (内置)
- **测试框架**: `flutter_test` (Widget 测试) + `mockito` / `mocktail` (Mock)
- **集成测试**: `integration_test` (Flutter 官方)

## 项目结构 (Layout)

采用 Feature-First 分层架构，每个功能模块自包含：

- `project_name/`
  - `lib/`
    - `main.dart` (应用入口)
    - `app.dart` (MaterialApp / GoRouter 配置)
    - `core/` (核心基础设施)
      - `constants/` (常量定义)
      - `theme/` (主题配置)
      - `utils/` (工具类)
      - `extensions/` (Dart 扩展方法)
      - `network/` (Dio 配置、拦截器)
      - `storage/` (本地存储封装)
      - `widgets/` (通用 Widget 组件)
    - `features/` (功能模块，Feature-First)
      - `auth/` (示例：认证模块)
        - `data/`
          - `datasources/` (远程/本地数据源)
          - `models/` (Freezed 数据模型)
          - `repositories/` (Repository 实现)
        - `domain/`
          - `entities/` (领域实体)
          - `repositories/` (Repository 抽象接口)
        - `presentation/`
          - `pages/` (页面 Widget)
          - `widgets/` (模块内 Widget)
          - `providers/` (Riverpod Providers)
    - `l10n/` (国际化)
  - `test/`
    - `unit/` (纯 Dart 单元测试)
    - `widget/` (Widget 测试)
    - `helpers/` (测试辅助工具)
  - `integration_test/` (集成测试)
  - `assets/` (静态资源)
    - `images/`
    - `fonts/`
    - `icons/`
  - `pubspec.yaml` (依赖配置)
  - `pubspec.lock` (锁定文件，严禁删除)
  - `analysis_options.yaml` (Lint 规则)
  - `build.yaml` (build_runner 配置)
  - `l10n.yaml` (国际化配置)
  - `.gitignore`
  - `README.md`

## 初始化工作流

当初始化一个新的 Flutter 项目时，按以下顺序执行：

- 创建 Flutter 项目：
  - 命令：`flutter create --org com.example --platforms ios,android project_name`
- 清理默认生成的示例代码。
- 创建 Feature-First 目录结构 (`lib/core/`, `lib/features/`)。
- 添加核心依赖：
  - 命令：`flutter pub add flutter_riverpod riverpod_annotation go_router dio freezed_annotation json_annotation`
- 添加开发依赖：
  - 命令：`flutter pub add --dev riverpod_generator build_runner freezed json_serializable mockito build_runner custom_lint riverpod_lint`
- 配置 `analysis_options.yaml`。
- 创建 `.gitignore`：
  - 必须包含 `.dart_tool/`, `.packages`, `build/`, `*.g.dart`, `*.freezed.dart`, `.flutter-plugins`, `.flutter-plugins-dependencies`, `*.iml`, `.idea/`。

## 配置标准

### pubspec.yaml 核心结构

```yaml
name: project_name
description: A Flutter application.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.3.0 <4.0.0'
  flutter: '>=3.19.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  go_router: ^14.0.0
  dio: ^5.4.0
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  mockito: ^5.4.0
  riverpod_lint: ^2.3.0
```

### analysis_options.yaml

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  strict-casts: true
  strict-inference: true
  strict-raw-types: true
  errors:
    missing_return: error
    missing_required_param: error
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"

linter:
  rules:
    - always_declare_return_types
    - annotate_overrides
    - avoid_empty_else
    - avoid_print
    - avoid_relative_lib_imports
    - prefer_const_constructors
    - prefer_const_declarations
    - prefer_final_fields
    - prefer_final_locals
    - require_trailing_commas
    - sort_constructors_first
    - unawaited_futures
```

## 开发指令集

Agent 必须使用以下命令执行开发任务：

- **添加依赖**：
  - `flutter pub add <package_name>`
- **运行应用**：
  - `flutter run`
- **运行代码生成** (Freezed / json_serializable / Riverpod)：
  - `dart run build_runner build --delete-conflicting-outputs`
- **监听代码生成** (开发时)：
  - `dart run build_runner watch --delete-conflicting-outputs`
- **运行单元测试**：
  - `flutter test`
- **运行指定测试文件**：
  - `flutter test test/unit/xxx_test.dart`
- **运行集成测试**：
  - `flutter test integration_test/`
- **格式化代码**：
  - `dart format .`
- **静态分析**：
  - `flutter analyze`
- **检查过时依赖**：
  - `flutter pub outdated`
- **构建 APK**：
  - `flutter build apk --release`
- **构建 iOS**：
  - `flutter build ios --release`

## 编码规范

- **Null Safety**：全面启用，严禁使用 `!` 强制解包，除非有充分理由并添加注释。
- **不可变优先**：
  - 数据模型使用 `Freezed` 生成不可变类。
  - Widget 参数使用 `final` 声明。
  - 优先使用 `const` 构造函数。
- **命名规范**：
  - 类名：`PascalCase` (如 `UserProfilePage`)
  - 文件名：`snake_case` (如 `user_profile_page.dart`)
  - 变量/函数：`camelCase` (如 `getUserProfile`)
  - 常量：`camelCase` 带 `k` 前缀 (如 `kDefaultPadding`) 或 `lowerCamelCase`
  - Provider：`camelCase` + `Provider` 后缀 (如 `userProfileProvider`)
- **Widget 拆分**：
  - 单个 Widget 的 `build` 方法不超过 80 行。
  - 超过时必须拆分为子 Widget 或提取为独立方法。
  - 优先使用 `StatelessWidget`，仅在必要时使用 `StatefulWidget`。
- **状态管理**：
  - 使用 Riverpod 的 `@riverpod` 注解声明 Provider。
  - 区分 `ref.watch` (响应式) 和 `ref.read` (一次性读取)。
  - 严禁在 `build` 方法中使用 `ref.read`，必须使用 `ref.watch`。

## 错误处理

- 网络请求使用 `Dio` 拦截器统一处理错误，封装为 `AppException`。
- 使用 `AsyncValue` (Riverpod) 处理异步状态 (loading / data / error)。
- 严禁使用裸露的 `catch` 语句，必须捕获具体异常类型。
- 在 `main.dart` 中配置 `FlutterError.onError` 和 `PlatformDispatcher.instance.onError` 捕获全局异常。
- 生产环境集成 Crashlytics 或 Sentry 进行错误上报。

## 测试规范

- **单元测试**：测试 Repository、Service、Provider 的业务逻辑。
- **Widget 测试**：使用 `WidgetTester` 测试 UI 组件的渲染和交互。
- **Mock**：使用 `mockito` 的 `@GenerateMocks` 注解生成 Mock 类。
- **Provider 测试**：使用 `ProviderContainer` 进行隔离测试。
- 测试文件命名：与源文件对应，后缀 `_test.dart` (如 `user_service_test.dart`)。
