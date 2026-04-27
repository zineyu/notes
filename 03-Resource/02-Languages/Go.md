# Golang 最佳实践

> 参考来源：[Effective Go](https://go.dev/doc/effective_go)、[Go Style Guide - Google](https://google.github.io/styleguide/go/guide.html)、[Standard Go Project Layout](https://github.com/golang-standards/project-layout)（55K+ stars）

---

## 核心原则

### 1. 简洁优于巧妙（Simplicity Over Cleverness）
以最简单的方式实现目标。Go 偏爱直接的解决方案而非抽象、巧妙的方案。如果需要复杂代码，必须充分记录理由。

### 2. 组合优于继承（Composition Over Inheritance）
使用接口和结构体嵌入实现代码复用。Go 没有继承；设计小而专注的接口（通常是单方法接口，使用 `-er` 后缀如 `Reader`、`Writer`）。

### 3. 显式错误处理（Explicit Error Handling）
立即显式检查错误。除非有意忽略，否则不要使用 `_` 忽略返回的错误。错误处理应该是可见的，而不是隐藏在异常中。

### 4. 通过 Goroutine 和 Channel 实现并发（Concurrency via Goroutines and Channels）
通过通信共享内存，不要通过共享内存通信。使用 goroutine 配合 channel 进行协调。始终考虑竞态条件。

### 5. 面向包的设计（Package-Oriented Design）
包是主要的组织单元。保持包名简短、有意义且小写。使用 `internal/` 存放不应被外部导入的实现细节。

---

## 项目架构/目录规范

### 推荐标准布局（参考 golang-standards/project-layout）

```text
myproject/
├── cmd/                    # 主应用程序
│   ├── api/
│   │   └── main.go         # HTTP 服务器入口（精简）
│   └── worker/
│       └── main.go         # 后台工作者入口
├── internal/               # 私有应用代码
│   ├── domain/             # 业务逻辑、实体
│   ├── service/            # 应用服务
│   ├── repository/         # 数据访问接口 + 实现
│   └── handler/            # HTTP 处理器
├── pkg/                    # 公共库代码（可选）
│   ├── httputil/
│   └── validator/
├── api/                    # API 定义（OpenAPI、protobuf）
├── configs/                # 配置文件
├── migrations/             # 数据库迁移
├── scripts/                # 构建和部署脚本
├── go.mod
└── go.sum
```

### 关键规则
- `cmd/` 包含精简的 `main.go` 文件，仅负责依赖注入
- `internal/` 是编译器强制私有的（不能被外部模块导入）
- 从小开始——单个 `main.go` + `go.mod` 对于小项目也是有效的

---

## 标准代码示例

### 示例 1：显式错误处理

```go
// Good: 立即检查错误，处理或传播
func readConfig(path string) (string, error) {
    f, err := os.Open(path)
    if err != nil {
        return "", fmt.Errorf("open config: %w", err)
    }
    defer f.Close()  // 确保清理

    var buf bytes.Buffer
    if _, err := buf.ReadFrom(f); err != nil {
        return "", fmt.Errorf("read config: %w", err)
    }
    return buf.String(), nil
}

// Good: 错误处理减少嵌套
data, err := readConfig("app.yaml")
if err != nil {
    log.Fatal(err)
}
// 使用 data...
```

### 示例 2：接口设计

```go
// Good: 小而专注的接口
type Storer interface {
    Store(key string, value []byte) error
    Retrieve(key string) ([]byte, error)
}

// Good: 构造函数（Go 惯用法）
type FileStore struct {
    basePath string
}

func NewFileStore(path string) (*FileStore, error) {
    if err := os.MkdirAll(path, 0750); err != nil {
        return nil, err
    }
    return &FileStore{basePath: path}, nil
}
```

---

## 常见反模式

### 1. 忽略错误

```go
// BAD: 静默忽略错误
f, _ := os.Open("file.txt")  // 如果失败，f 为 nil
f.Close()

// GOOD: 始终检查错误
f, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer f.Close()
```

### 2. 使用 panic 处理正常错误

```go
// BAD: 对预期错误使用 panic
func findUser(id string) *User {
    user, err := db.GetUser(id)
    if err != nil {
        panic(err)  // 不要这样做！
    }
    return user
}

// GOOD: 正常返回错误
func findUser(id string) (*User, error) {
    user, err := db.GetUser(id)
    if err != nil {
        return nil, fmt.Errorf("get user %s: %w", id, err)
    }
    return user, nil
}
```

### 3. 接口污染（过早抽象）

```go
// BAD: 只有一个实现的接口
type UserService interface {
    CreateUser(name string) (*User, error)
    GetUser(id string) (*User, error)
    DeleteUser(id string) error
    // ... 还有 15+ 个方法
}

// GOOD: 具体类型，需要时在调用点提取接口
type UserService struct {
    db *sql.DB
}

func (s *UserService) CreateUser(name string) (*User, error) { /* ... */ }
```

---

## 参考资源

- [Effective Go](https://go.dev/doc/effective_go)（官方）
- [Go Style Guide - Google](https://google.github.io/styleguide/go/guide.html)（Google）
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)（55K+ stars）
- [How to Write Go Code](https://go.dev/doc/code)（官方）

---

## 相关笔记

- [[后端架构]] — Go 在后端架构中的典型应用场景
