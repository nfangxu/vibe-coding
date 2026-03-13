# React 开发规范

> 适用于 React 18 + TypeScript + Vite 项目，在 AI 编辑器中使用时可将本文件内容配置为规则。

请参考 [React Cursor 规则](../.cursor/rules/react.mdc) 获取完整规范。

## 核心原则

1. **函数组件**：始终使用函数组件，不使用类组件
2. **TanStack Query**：使用 TanStack Query 管理服务端状态，而非手动 useEffect + useState
3. **类型安全**：为所有 Props、Hook 返回值添加 TypeScript 类型
4. **状态分层**：服务端状态用 Query，全局 UI 状态用 Zustand，本地状态用 useState

## 推荐工具

| 工具 | 用途 |
|------|------|
| `vite` | 构建工具 |
| `react-router-dom` | 路由 |
| `@tanstack/react-query` | 服务端状态管理 |
| `zustand` | 客户端全局状态 |
| `react-hook-form` + `zod` | 表单验证 |
| `vitest` + `@testing-library/react` | 单元/集成测试 |
| `playwright` | E2E 测试 |
