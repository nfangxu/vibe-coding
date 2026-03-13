# vibe-coding

Vibe Coding 常用的一些规则示例，针对不同的编辑器 Trae / Cursor / Windsurf / JetBrains 等。

以上规则仅表示个人的常用习惯，不要犟。

## 目录结构

```
.
├── .cursor/
│   └── rules/              # Cursor 编辑器规则（.mdc 格式）
│       ├── golang.mdc      # Go 语言开发规范
│       ├── vue3.mdc        # Vue 3 开发规范
│       ├── typescript.mdc  # TypeScript 开发规范
│       ├── react.mdc       # React 开发规范
│       ├── python.mdc      # Python 开发规范
│       ├── restful-api.mdc # RESTful API 设计规范
│       └── git-commit.mdc  # Git 提交规范
└── rules/                  # 通用规范文档（Markdown 格式）
    ├── golang.md
    ├── vue3.md
    ├── typescript.md
    ├── react.md
    ├── python.md
    └── restful-api.md
```

## 规则列表

| 规则 | 说明 | Cursor 规则 | 通用文档 |
|------|------|-------------|----------|
| **Go 语言** | 命名规范、错误处理、并发、测试 | [golang.mdc](.cursor/rules/golang.mdc) | [golang.md](rules/golang.md) |
| **Vue 3** | Composition API、Pinia、路由、组件规范 | [vue3.mdc](.cursor/rules/vue3.mdc) | [vue3.md](rules/vue3.md) |
| **TypeScript** | 类型定义、泛型、类型守卫、模块化 | [typescript.mdc](.cursor/rules/typescript.mdc) | [typescript.md](rules/typescript.md) |
| **React** | Hooks、状态管理、表单、性能优化 | [react.mdc](.cursor/rules/react.mdc) | [react.md](rules/react.md) |
| **Python** | 类型注解、Pydantic、FastAPI、测试 | [python.mdc](.cursor/rules/python.mdc) | [python.md](rules/python.md) |
| **RESTful API** | URL 设计、HTTP 方法、状态码、分页、安全 | [restful-api.mdc](.cursor/rules/restful-api.mdc) | [restful-api.md](rules/restful-api.md) |
| **Git 提交** | Conventional Commits、分支规范、CR 清单 | [git-commit.mdc](.cursor/rules/git-commit.mdc) | — |

## 如何使用

### Cursor

`.cursor/rules/` 目录下的 `.mdc` 文件会被 Cursor 自动识别。每个规则文件的 frontmatter 中配置了：

- `description`：规则描述，Cursor Agent 会根据描述判断何时应用此规则
- `globs`：文件匹配模式，匹配时自动应用规则
- `alwaysApply`：是否始终应用（`true` 表示对所有文件生效）

### Windsurf

将 `.cursor/rules/` 目录下的规则内容复制到 Windsurf 的 Rules 配置中，或在项目根目录创建 `.windsurfrules` 文件。

### Trae / JetBrains AI

将对应规则文件的内容添加到编辑器的 AI 规则/系统提示配置中。

### 通用

`rules/` 目录下的 Markdown 文件可直接阅读或复制到任何支持自定义规则的 AI 工具中。

## 贡献

欢迎提交 PR 添加新的规则，例如：

- 更多语言：Java、Rust、Kotlin、Swift
- 更多框架：Next.js、Nuxt、NestJS、Spring Boot
- 特定场景：微服务、数据库设计、API 设计、安全规范
