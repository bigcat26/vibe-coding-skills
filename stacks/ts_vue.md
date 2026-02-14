---
name: stack-ts-vue
description: TypeScript/Node.js/Vue 前端开发技术栈标准。使用 Vue 3 + Vite 构建现代化前端应用，集成 Pinia (状态管理)、Vue Router、ESLint + Prettier (代码规范) 和 Vitest (测试)。
version: 1.0.0
tags: [typescript, nodejs, vue, vite, pinia, vitest]
---

# TypeScript / Node.js / Vue 前端开发技术栈规范

本文档定义了 Vibe Coding 体系下 TypeScript + Vue 前端项目的强制性技术栈和开发标准。核心理念是类型安全、组合式 API 和组件驱动开发。

## 工具链标准

Agent 必须使用以下工具，除非用户明确指定其他替代方案：

- **运行时**: `Node.js` (>= 20 LTS)
- **包管理器**: `pnpm` (替代 npm, yarn)
- **构建工具**: `Vite` (>= 5.x)
- **前端框架**: `Vue 3` (Composition API + `<script setup>`)
- **语言**: `TypeScript` (严格模式)
- **状态管理**: `Pinia`
- **路由**: `Vue Router 4`
- **HTTP 客户端**: `Axios` (封装拦截器)
- **UI 组件库**: `Element Plus` 或 `Ant Design Vue` 或 `Naive UI` (按需选择)
- **CSS 方案**: `UnoCSS` 或 `TailwindCSS`
- **图标**: `Iconify` (unplugin-icons)
- **代码检查**: `ESLint` (eslint-plugin-vue + @typescript-eslint)
- **代码格式化**: `Prettier`
- **测试框架**: `Vitest` (单元测试) + `Playwright` 或 `Cypress` (E2E 测试)
- **Git Hooks**: `husky` + `lint-staged`

## 项目结构 (Layout)

- `project_name/`
  - `src/`
    - `main.ts` (应用入口)
    - `App.vue` (根组件)
    - `api/` (API 请求封装)
      - `index.ts` (Axios 实例配置)
      - `modules/` (按模块拆分的 API 定义)
    - `assets/` (静态资源)
      - `styles/` (全局样式)
      - `images/`
    - `components/` (通用组件)
      - `common/` (基础通用组件)
      - `business/` (业务通用组件)
    - `composables/` (组合式函数 / Hooks)
    - `directives/` (自定义指令)
    - `layouts/` (布局组件)
    - `pages/` 或 `views/` (页面组件)
    - `router/` (路由配置)
      - `index.ts`
      - `modules/` (按模块拆分的路由)
    - `stores/` (Pinia 状态管理)
    - `types/` (TypeScript 类型定义)
    - `utils/` (工具函数)
    - `plugins/` (Vue 插件)
  - `public/` (静态公共资源)
  - `tests/`
    - `unit/` (单元测试)
    - `e2e/` (E2E 测试)
  - `index.html`
  - `package.json`
  - `pnpm-lock.yaml` (锁定文件，严禁删除)
  - `vite.config.ts`
  - `tsconfig.json`
  - `tsconfig.app.json`
  - `tsconfig.node.json`
  - `.eslintrc.cjs` 或 `eslint.config.js`
  - `.prettierrc`
  - `.env` / `.env.development` / `.env.production`
  - `.gitignore`
  - `README.md`

## 初始化工作流

当初始化一个新的 Vue 项目时，按以下顺序执行：

- 创建 Vite + Vue + TypeScript 项目：
  - 命令：`pnpm create vue@latest project_name`
  - 选项：启用 TypeScript、Vue Router、Pinia、ESLint + Prettier、Vitest、Playwright/Cypress。
- 安装依赖：
  - 命令：`pnpm install`
- 添加核心依赖：
  - 命令：`pnpm add axios`
  - 命令：`pnpm add -D unplugin-auto-import unplugin-vue-components`
- 添加 UI 组件库 (按需)：
  - 命令：`pnpm add element-plus` 或 `pnpm add ant-design-vue` 或 `pnpm add naive-ui`
- 配置 Git Hooks：
  - 命令：`pnpm add -D husky lint-staged`
  - 命令：`pnpm exec husky init`
- 创建 `.gitignore`：
  - 必须包含 `node_modules/`, `dist/`, `.env.local`, `.env.*.local`, `*.log`, `.vite/`, `coverage/`。

## 配置标准

### vite.config.ts

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'

export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: ['vue', 'vue-router', 'pinia'],
      dts: 'src/auto-imports.d.ts',
    }),
    Components({
      dts: 'src/components.d.ts',
    }),
  ],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
})
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "jsx": "preserve",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "types": ["vite/client"]
  }
}
```

### ESLint 配置 (eslint.config.js - Flat Config)

```javascript
import pluginVue from 'eslint-plugin-vue'
import tseslint from 'typescript-eslint'

export default [
  ...pluginVue.configs['flat/recommended'],
  ...tseslint.configs.recommended,
  {
    rules: {
      'vue/multi-word-component-names': 'off',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'warn',
    },
  },
]
```

### Prettier 配置 (.prettierrc)

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "vueIndentScriptAndStyle": true
}
```

## 开发指令集

Agent 必须使用以下命令执行开发任务：

- **安装依赖**：
  - `pnpm install`
- **启动开发服务器**：
  - `pnpm dev`
- **构建生产版本**：
  - `pnpm build`
- **预览生产构建**：
  - `pnpm preview`
- **运行单元测试**：
  - `pnpm test:unit` 或 `pnpm vitest`
- **运行 E2E 测试**：
  - `pnpm test:e2e`
- **格式化代码**：
  - `pnpm prettier --write .`
- **检查代码**：
  - `pnpm eslint . --fix`
- **类型检查**：
  - `pnpm vue-tsc --noEmit`

## 编码规范

- **Composition API**：全面使用 `<script setup lang="ts">`，严禁使用 Options API。
- **TypeScript 严格模式**：启用 `strict: true`，所有变量和函数必须有明确类型。
- **命名规范**：
  - 组件文件名：`PascalCase.vue` (如 `UserProfile.vue`)
  - 组合式函数：`use` 前缀 + `camelCase` (如 `useUserProfile.ts`)
  - Store：`use` 前缀 + `camelCase` + `Store` 后缀 (如 `useUserStore.ts`)
  - 类型/接口：`PascalCase`，接口不加 `I` 前缀 (如 `UserProfile`, 非 `IUserProfile`)
  - 常量：`UPPER_SNAKE_CASE` (如 `API_BASE_URL`)
  - 事件：`on` 前缀 + `PascalCase` (如 `onSubmitForm`)
- **组件规范**：
  - Props 必须使用 `defineProps<T>()` 泛型定义，包含类型和默认值。
  - Emits 必须使用 `defineEmits<T>()` 泛型定义。
  - 单个 `.vue` 文件不超过 300 行，超过时拆分为子组件。
  - 模板中严禁复杂表达式，提取为 `computed` 或方法。
- **状态管理**：
  - 使用 Pinia 的 Setup Store 语法 (组合式写法)。
  - Store 中只存放跨组件共享的状态，组件内部状态使用 `ref` / `reactive`。
  - 异步操作放在 Store 的 actions 中，组件不直接调用 API。
- **API 封装**：
  - Axios 实例统一配置拦截器 (请求 token 注入、响应错误处理)。
  - 每个模块的 API 独立文件，返回类型明确。

## 错误处理

- Axios 响应拦截器统一处理 HTTP 错误码 (401 跳转登录、403 权限提示、500 通用错误)。
- 组件级错误使用 `onErrorCaptured` 生命周期钩子。
- 全局错误使用 `app.config.errorHandler` 捕获。
- 异步操作使用 `try-catch`，并向用户展示友好的错误提示 (Toast / Message)。
- 严禁静默吞掉错误，至少记录到 `console.error` 或上报到监控平台。

## 测试规范

- **单元测试**：使用 `Vitest` 测试 composables、stores、utils。
- **组件测试**：使用 `@vue/test-utils` + `Vitest` 测试组件渲染和交互。
- **E2E 测试**：使用 `Playwright` 或 `Cypress` 测试关键用户流程。
- **Mock**：使用 `vi.mock()` 模拟模块，`vi.fn()` 模拟函数。
- 测试文件命名：与源文件对应，后缀 `.test.ts` 或 `.spec.ts`。
- 测试覆盖率目标：>= 80%，配置 `vitest.config.ts` 中的 `coverage`。
