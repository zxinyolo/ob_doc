[TOC]

## 一、 项目结构 (Project Layout)

一个良好、统一的项目结构能够极大地提高代码的可读性和可维护性。社区推荐遵循 [Standard Go Project Layout](https://github.com/golang-standards/project-layout)。

### 核心目录解析

```
my-project/
├── /cmd/          # 项目的主入口
│   └── my-app/    # 应用名
│       └── main.go
├── /internal/     # 内部私有代码，外部无法导入
│   ├── /app/      # 应用核心逻辑 (handler, service, repository)
│   ├── /pkg/      # 项目内部共享的包
│   └── /model/    # 数据模型 (structs)
├── /pkg/          # 可被外部项目导入的公共代码
├── /api/          # API 定义文件 (e.g., .proto, .swagger.json)
├── /configs/      # 配置文件 (config.yaml)
├── /scripts/      # 脚本文件 (build, deploy)
├── go.mod         # Go 模块文件
└── go.sum         # 依赖校验和
```

-   **/cmd**: 存放项目的主程序入口。每个子目录对应一个可执行文件。
-   **/internal**: **核心亮点**。此目录下的代码只能被该项目内部的代码导入，Go 编译器会强制执行这个规则。这是存放项目核心业务逻辑的最佳位置，可以有效防止项目代码被意外地依赖。
-   **/pkg**: 存放可以被外部项目安全导入的公共库代码。如果你的代码没有必要被外部引用，优先放在 `/internal`。
-   **/api**: 存放 API 契约文件，如 Protobuf 定义、Swagger/OpenAPI JSON 文件等。
-   **/configs**: 存放配置文件模板或默认配置。

## 二、 Go Modules 与依赖管理

Go Modules 是 Go 1.11 引入的官方依赖管理系统，解决了 `GOPATH` 模式下的许多问题。

### 2.1 `go.mod` 文件

`go.mod` 是模块的“身份证”，定义了模块路径和依赖关系。

```go
module github.com/my-user/my-project // 模块的导入路径

go 1.18 // Go 版本

require (
    github.com/gin-gonic/gin v1.8.1
    github.com/spf13/viper v1.12.0
)

// replace github.com/some/lib v1.2.3 => ../some/lib // 替换依赖为本地路径，常用于调试
// exclude github.com/some/bad-lib v1.0.0 // 排除某个版本的依赖
```

### 2.2 `go.sum` 文件

`go.sum` 记录了每个直接和间接依赖的特定版本的加密哈希值。它的作用是**保证构建的可重现性和安全性**。当你构建项目时，Go 会下载依赖并计算其哈希，如果与 `go.sum` 中的记录不匹配，构建就会失败，这可以防止依赖被篡改。

### 2.3 常用命令

-   `go mod tidy`: **最常用的命令**。它会添加缺失的依赖，并移除代码中未使用的依赖。
-   `go mod vendor`: 将所有依赖项复制到项目根目录下的 `vendor` 文件夹中。如果使用 `go build -mod=vendor`，编译器将只使用 `vendor` 中的依赖，实现离线构建。
-   `go mod why [package]`: 解释为什么需要某个特定的依赖包。

## 三、 错误处理哲学

Go 语言通过显式的 `error` 返回值来处理错误，这使得错误处理成为编程的一等公民。

### 3.1 哨兵错误 (Sentinel Errors)

通过定义一个导出的错误变量，让调用者可以直接使用 `==` 来判断是否是某个特定的、预期的错误。

```go
import "errors"

// ErrUserNotFound 是一个预定义的错误
var ErrUserNotFound = errors.New("user not found")

func GetUser(id int) (*User, error) {
    if id <= 0 {
        return nil, ErrUserNotFound
    }
    // ... a
    return &User{}, nil
}

func main() {
    _, err := GetUser(0)
    if err == ErrUserNotFound {
        fmt.Println("This is a user not found error.")
    }
}
```
**缺点**: 调用者与包之间产生了强耦合。

### 3.2 错误类型 (Error Types)

通过定义一个实现了 `error` 接口的自定义类型，可以携带更丰富的上下文信息。

```go
type MyError struct {
    Code    int
    Message string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}

func someOperation() error {
    return &MyError{Code: 500, Message: "Internal server error"}
}
```

### 3.3 错误包装 (Error Wrapping)

自 Go 1.13 起，`fmt.Errorf` 支持 `%w` 动词来“包装”一个错误，保留原始错误的上下文。这解决了在调用链中错误信息丢失的问题。

-   `%w`: 包装错误。
-   `errors.Is()`: 判断一个错误是否是某个特定的目标错误（会检查整个错误链）。
-   `errors.As()`: 判断错误链中是否有某个特定的错误类型，并将其赋值给一个变量。

```go
func dataAccess() error {
    return ErrUserNotFound // 原始错误
}

func businessLogic() error {
    err := dataAccess()
    if err != nil {
        // 使用 %w 包装错误，添加上下文信息
        return fmt.Errorf("business logic failed: %w", err)
    }
    return nil
}

func main() {
    err := businessLogic()
    // 使用 errors.Is 来检查最根本的原因
    if errors.Is(err, ErrUserNotFound) {
        fmt.Println("We know the root cause is user not found.")
        // 输出: business logic failed: user not found
        fmt.Println(err)
    }
}
```

## 四、 配置管理

推荐使用 [Viper](https://github.com/spf13/viper) 库，它可以无缝地从多种来源（文件、环境变量、命令行参数、远程 K/V 存储）读取配置。

**示例 (`config.yaml`)**:
```yaml
server:
  port: 8080
database:
  host: "localhost"
  user: "root"
```

**Go 代码**:
```go
import "github.com/spf13/viper"

func main() {
    viper.SetConfigName("config") // 配置文件名 (无扩展名)
    viper.SetConfigType("yaml")   // 配置文件类型
    viper.AddConfigPath("./configs") // 配置文件路径

    if err := viper.ReadInConfig(); err != nil {
        panic(fmt.Errorf("fatal error config file: %w", err))
    }

    port := viper.GetInt("server.port") // 8080
    dbUser := viper.GetString("database.user") // "root"
}
```

## 五、 日志 (Logging)

良好的日志实践对于调试和监控至关重要。推荐使用**结构化日志**。

-   **结构化日志**: 将日志信息以 JSON 或 key-value 格式输出，便于机器解析、索引和查询（如通过 ELK 或 Loki 系统）。
-   **推荐库**: 
    -   [Logrus](https://github.com/sirupsen/logrus): 非常流行，API 友好。
    -   [Zap](https://github.com/uber-go/zap): 由 Uber 开发，以极高的性能著称。

**Zap 示例**:
```go
import "go.uber.org/zap"

func main() {
    logger, _ := zap.NewProduction()
    defer logger.Sync() // 确保所有缓冲的日志都被写入

    userID := 123
    orderID := "abc-456"

    // 日志中包含了结构化的字段
    logger.Info("User placed an order",
        zap.Int("userID", userID),
        zap.String("orderID", orderID),
    )
    // 输出 (JSON):
    // {"level":"info","ts":166... ,"caller":"...","msg":"User placed an order","userID":123,"orderID":"abc-456"}
}
```

## 六、 测试 (Testing)

Go 内置了强大的测试框架 `testing`。

-   **单元测试**: 文件名以 `_test.go` 结尾，函数以 `TestXxx` 开头。
-   **基准测试**: 函数以 `BenchmarkXxx` 开头，用于性能测试。
-   **示例测试**: 函数以 `ExampleXxx` 开头，既是文档说明，也是可执行的测试。

**示例 (`math_test.go`)**:
```go
package mymath

import "testing"

func Add(a, b int) int {
    return a + b
}

// 单元测试
func TestAdd(t *testing.T) {
    if Add(1, 2) != 3 {
        t.Error("1 + 2 should be 3")
    }
}

// 基准测试
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(1, 2)
    }
}

// 运行测试: go test
// 运行基准测试: go test -bench=.
```
