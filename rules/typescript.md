# TypeScript 开发规范

> 适用于所有 TypeScript 项目，在 AI 编辑器中使用时可将本文件内容配置为规则。

请参考 [TypeScript Cursor 规则](../.cursor/rules/typescript.mdc) 获取完整规范。

## 核心原则

1. **严格模式**：始终开启 `"strict": true`
2. **类型安全**：避免 `any`，使用 `unknown` + 类型守卫
3. **命名导出**：优先使用命名导出而非默认导出
4. **最小化断言**：避免 `as` 类型断言，使用类型守卫代替

## 推荐工具

| 工具 | 用途 |
|------|------|
| `typescript` | 语言核心 |
| `tsx` | 直接运行 TS 文件 |
| `tsc --noEmit` | 类型检查 |
| `eslint` + `typescript-eslint` | 代码规范 |
| `zod` | 运行时类型验证 |
