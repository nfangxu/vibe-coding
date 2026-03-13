# Python 开发规范

> 适用于 Python 3.10+ 项目，在 AI 编辑器中使用时可将本文件内容配置为规则。

请参考 [Python Cursor 规则](../.cursor/rules/python.mdc) 获取完整规范。

## 核心原则

1. **类型注解**：为所有函数参数和返回值添加类型注解，开启 `mypy --strict`
2. **现代工具链**：使用 `uv` 管理依赖，`ruff` 进行格式化和 lint
3. **Pydantic 验证**：使用 Pydantic v2 进行数据验证
4. **异步优先**：Web 服务使用 async/await，利用 FastAPI + asyncpg

## 推荐工具

| 工具 | 用途 |
|------|------|
| `uv` | 依赖管理与虚拟环境 |
| `ruff` | 格式化 + Lint（替代 black + flake8 + isort）|
| `mypy` | 静态类型检查 |
| `pytest` + `pytest-asyncio` | 测试框架 |
| `pydantic` | 数据验证 |
| `fastapi` | Web 框架 |
| `structlog` | 结构化日志 |
