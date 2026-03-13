# Go 语言开发规范

> 适用于所有 Go 项目，在 AI 编辑器中使用时可将本文件内容配置为规则。

## 代码风格

- 所有代码必须通过 `gofmt` 格式化，提交前运行 `goimports` 整理导入
- 使用 `golangci-lint` 进行静态检查，遵循 lint 报告中的所有警告
- 行长度建议不超过 120 个字符
- 使用 Tab 缩进，不使用空格

## 命名规范

- 包名使用小写单词，不使用下划线或驼峰：`package userservice`
- 导出的标识符使用 PascalCase：`type UserService struct`
- 未导出的标识符使用 camelCase：`func parseToken()`
- 接口名以行为命名，单方法接口通常以 `-er` 结尾：`type Reader interface`
- 常量使用 PascalCase（导出）或 camelCase（未导出）
- 避免在名称中重复包名：使用 `user.Service` 而非 `user.UserService`
- 错误变量命名以 `Err` 开头：`var ErrNotFound = errors.New("not found")`

## 项目结构

```
project/
├── cmd/              # 可执行程序入口
│   └── server/
│       └── main.go
├── internal/         # 私有应用代码（不对外暴露）
│   ├── handler/      # HTTP 处理器
│   ├── service/      # 业务逻辑层
│   ├── repository/   # 数据访问层
│   └── model/        # 数据模型
├── pkg/              # 可被外部使用的公共库
├── api/              # API 定义（protobuf / OpenAPI）
├── configs/          # 配置文件
├── scripts/          # 构建/部署脚本
├── go.mod
└── go.sum
```

## 错误处理

- 始终检查并处理错误，不要使用 `_` 忽略关键错误
- 使用 `fmt.Errorf("操作失败: %w", err)` 包装错误以保留堆栈信息
- 在顶层统一处理错误并记录日志，底层只负责向上传递
- 自定义错误类型实现 `error` 接口，使用 `errors.Is` / `errors.As` 进行判断

```go
// 推荐
if err := doSomething(); err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// 自定义错误
type NotFoundError struct {
    ID string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("resource %s not found", e.ID)
}
```

## 接口设计

- 在使用方定义接口，而非实现方
- 保持接口小而专一（Interface Segregation Principle）
- 优先使用组合而非继承

```go
// 推荐：小接口，在使用方定义
type UserReader interface {
    GetUser(ctx context.Context, id string) (*User, error)
}

type UserWriter interface {
    CreateUser(ctx context.Context, user *User) error
    UpdateUser(ctx context.Context, user *User) error
}
```

## 并发编程

- 使用 `context.Context` 控制 goroutine 生命周期，作为函数第一个参数
- 使用 `sync.WaitGroup` 等待 goroutine 完成
- 使用 channel 进行 goroutine 间通信，使用 mutex 保护共享状态
- 避免 goroutine 泄漏，确保每个 goroutine 都有退出路径

```go
// 推荐：使用 errgroup 管理并发任务
func fetchAll(ctx context.Context, ids []string) ([]*User, error) {
    g, ctx := errgroup.WithContext(ctx)
    results := make([]*User, len(ids))
    
    for i, id := range ids {
        i, id := i, id // 捕获循环变量
        g.Go(func() error {
            user, err := fetchUser(ctx, id)
            if err != nil {
                return err
            }
            results[i] = user
            return nil
        })
    }
    
    if err := g.Wait(); err != nil {
        return nil, err
    }
    return results, nil
}
```

## 测试规范

- 测试文件以 `_test.go` 结尾，与被测文件放在同一包
- 单元测试函数命名：`TestFunctionName_Scenario_ExpectedResult`
- 使用 `testify` 库进行断言：`assert.Equal(t, expected, actual)`
- 使用表格驱动测试（Table-Driven Tests）
- 使用 `t.Parallel()` 并行运行独立测试

```go
func TestGetUser_ValidID_ReturnsUser(t *testing.T) {
    t.Parallel()
    
    tests := []struct {
        name    string
        id      string
        want    *User
        wantErr bool
    }{
        {
            name: "valid user id",
            id:   "user-123",
            want: &User{ID: "user-123", Name: "Alice"},
        },
        {
            name:    "empty id returns error",
            id:      "",
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        tt := tt
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            got, err := GetUser(tt.id)
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

## 依赖注入

- 通过构造函数注入依赖，避免全局变量
- 使用 `wire` 或手动依赖注入

```go
type UserService struct {
    repo   UserRepository
    cache  Cache
    logger *slog.Logger
}

func NewUserService(repo UserRepository, cache Cache, logger *slog.Logger) *UserService {
    return &UserService{
        repo:   repo,
        cache:  cache,
        logger: logger,
    }
}
```

## 日志规范

- 使用 `log/slog`（Go 1.21+）进行结构化日志记录
- 日志级别：Debug < Info < Warn < Error
- 在函数入口记录关键参数，在错误处记录完整上下文

```go
slog.InfoContext(ctx, "user created",
    slog.String("user_id", user.ID),
    slog.String("email", user.Email),
)
```

## HTTP 服务

- 始终为 `http.Server` 设置超时：`ReadTimeout`、`WriteTimeout`、`IdleTimeout`
- 使用 `context` 进行请求取消和超时控制
- 统一的错误响应格式
- 使用中间件处理跨切面关注点（鉴权、日志、限流）

## 性能

- 优先使用 `strings.Builder` 进行字符串拼接
- 预分配切片容量：`make([]T, 0, capacity)`
- 使用 `sync.Pool` 复用临时对象
- 避免在热路径中进行内存分配
- 使用 `pprof` 进行性能分析

## 推荐工具

| 工具 | 用途 |
|------|------|
| `gofmt` / `goimports` | 代码格式化 |
| `golangci-lint` | 静态分析 |
| `go test -race` | 竞态检测 |
| `go vet` | 代码检查 |
| `wire` | 依赖注入 |
| `testify` | 测试断言 |
| `gomock` / `mockery` | Mock 生成 |
| `pprof` | 性能分析 |
