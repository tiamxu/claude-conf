---
name: go-project-init
description: 用于根据团队的《Go工程开发规范》，一键从零初始化一个新的Go语言Web/API项目骨架。当用户要求初始化Go新项目、创建新项目脚手架、或输入 /go-project-init 时触发。
disable-model-invocation: true
user-invocable: true
---

# 🛠️ Go 规范项目一键初始化助手

当你激活本技能时，你的角色是**"高级自动化平台工程师"**。你必须完全自主地调用系统工具（创建目录、写文件、执行命令），在当前空白目录下从零构建全套符合规范的 Go 工程脚手架。

请一次性、不折不扣地执行以下所有阶段。

---

## 📂 第一阶段：构建标准拓扑结构

请在当前目录下自主执行 `mkdir -p` 创建以下所有文件夹：
- `api/`
- `service/`
- `repo/`
- `model/`
- `types/`
- `middleware/`
- `config/`
- `routes/`
- `pkg/e/`
- `pkg/constants/`
- `pkg/utils/`
- `doc/`
- `sql/`

---

## 📝 第二阶段：全自动注入规范代码文件

### 1. pkg/e/error.go — 统一错误码

```go
package e

const (
	SUCCESS = 200

	// 客户端错误 (1xxxx)
	InvalidParams = 10001
	Unauthorized  = 10002

	// 服务端错误 (5xxxx)
	ERROR         = 50000
	InternalError = 50001

	// 业务错误 (2xxxx)
	ErrNotFound = 20001
)

var msgMap = map[int]string{
	SUCCESS:       "操作成功",
	ERROR:         "服务器内部错误",
	InvalidParams: "参数错误",
	Unauthorized:  "未授权",
	InternalError: "服务器内部错误",
	ErrNotFound:   "资源不存在",
}

func GetMsg(code int) string {
	if msg, ok := msgMap[code]; ok {
		return msg
	}
	return "未知错误"
}
```

### 2. api/common.go — 统一响应

```go
package api

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

type Response struct {
    Code    int         `json:"code"`
    Message string      `json:"message"`
    Data    interface{} `json:"data,omitempty"`
}

func Success(c *gin.Context, data interface{}) {
    c.JSON(http.StatusOK, Response{Code: 200, Message: "ok", Data: data})
}

func Error(c *gin.Context, httpStatus int, bizCode int, message string) {
    c.JSON(httpStatus, Response{Code: bizCode, Message: message})
}
```

### 3. config/config.go — 配置加载

```go
package config

import (
    "fmt"
    "os"

    "github.com/tiamxu/kit/http"
    "github.com/tiamxu/kit/log"
    "github.com/tiamxu/kit/sql"
    "github.com/tiamxu/kit/cache"
    "github.com/koding/multiconfig"
)

var (
    cfg        *Config
    configPath = "config/config.yaml"
)

type Config struct {
    ENV     string          `yaml:"env"`
    Log     log.Config     `yaml:"log"`
    HttpSrv http.ServerConfig `yaml:"http_srv"`
    DB      *sql.Config    `yaml:"db"`
    Cache   cache.Config   `yaml:"cache"`
}

func LoadConfig() *Config {
    cfg := new(Config)

    env := os.Getenv("ENV")
    if env == "" {
        env = "dev"
    }

    switch env {
    case "dev":
        configPath = "config/config-dev.yaml"
    case "test":
        configPath = "config/config-test.yaml"
    case "prod":
        configPath = "config/config-prod.yaml"
    }

    multiconfig.MustLoadWithPath(configPath, cfg)
    return cfg
}

func (c *Config) Initial() error {
    if err := log.InitLogger(&c.Log); err != nil {
        return fmt.Errorf("log init failed: %w", err)
    }

    if err := repo.Init(c.DB); err != nil {
        return fmt.Errorf("database init failed: %w", err)
    }

    log.Infof("config initialed, env: %s", c.ENV)
    return nil
}
}
```

### 4. config/config-dev.yaml — 开发环境配置

```yaml
env: dev
log:
  level: info
  type: stdout
  format: console
http_srv:
  address: ":8800"
  keep_alive: true
  read_timeout: 30s
  write_timeout: 30s
  body_limit: 8388608
db:
  driver: mysql
  host: "127.0.0.1"
  port: 3306
  database: "app"
  username: "${DB_USERNAME}"
  password: "${DB_PASSWORD}"
  max_idle_conns: 10
  max_open_conns: 50
  conn_max_lifetime: 3600
```

### 5. repo/init.go — 数据库初始化

```go
package repo

import (
    "context"
    "errors"
    "time"

    "github.com/tiamxu/kit/log"
    "github.com/tiamxu/kit/sql"
)

var DB *sql.DB

func Init(cfg *sql.Config) error {
    if cfg == nil {
        return errors.New("database config is nil")
    }

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    db, err := sql.Connect(cfg)
    if err != nil {
        log.Errorf("database connect failed: %v", err)
        return err
    }

    if err := db.PingContext(ctx); err != nil {
        log.Errorf("database ping failed: %v", err)
        return err
    }

    DB = db
    log.Infof("database initialized, driver: %s, host: %s:%d, max_open: %d",
        cfg.Driver, cfg.Host, cfg.Port, cfg.MaxOpenConns)
    return nil
}

func Close() error {
    if DB != nil {
        return DB.Close()
    }
    return nil
}

// IsNoRows 判断是否为"无数据"错误
func IsNoRows(err error) bool {
    return sql.IsNoRows(err)
}
```

**说明：**
- 使用 `sql.Connect(cfg)` 创建连接
- 使用 `sql.IsNoRows(err)` 判断无数据错误
- 使用 `DB.TransactCallback(fn)` 进行事务
- 使用 `PingContext` 验证连接，带超时控制

### 6. middleware/auth.go — JWT认证

```go
package middleware

import (
    "net/http"
    "strings"

    "github.com/gin-gonic/gin"
)

type Claims struct {
    UID int64 `json:"uid"`
}

func JWTAuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := strings.TrimPrefix(c.GetHeader("Authorization"), "Bearer ")
        if token == "" {
            c.JSON(http.StatusUnauthorized, gin.H{"code": 10002, "message": "missing token"})
            c.Abort()
            return
        }

        claims, err := parseToken(token)
        if err != nil {
            c.JSON(http.StatusUnauthorized, gin.H{"code": 10002, "message": "invalid token"})
            c.Abort()
            return
        }

        c.Set("uid", claims.UID)
        c.Next()
    }
}

func parseToken(token string) (*Claims, error) {
    return &Claims{UID: 1}, nil
}
```

### 7. routes/routes.go — 路由注册

```go
package routes

import (
    "github.com/gin-gonic/gin"
    "github.com/company/project/api"
    "github.com/company/project/middleware"
)

func InitRoutes(r *gin.Engine) {
    r.Use(gin.Recovery())

    r.GET("/health", func(c *gin.Context) {
        api.Success(c, gin.H{"status": "up"})
    })

    public := r.Group("/public")
    {
        // public endpoints
    }

    apiGroup := r.Group("/api")
    apiGroup.Use(middleware.JWTAuthMiddleware())
    {
        // api endpoints
    }
}
```

### 8. main.go — 程序入口

```go
package main

import (
    "os"
    "os/signal"
    "syscall"

    "github.com/company/project/config"
    "github.com/company/project/repo"
    "github.com/company/project/routes"
    "github.com/tiamxu/kit/http"
    "github.com/tiamxu/kit/log"
)

func main() {
    cfg := config.LoadConfig()

    if err := cfg.Initial(); err != nil {
        log.Fatalf("config initial failed: %v", err)
    }

    // 使用 kit/http 创建 Gin Engine（内置中间件：RequestID、AccessLog、CORS、ErrorHandler）
    router := http.NewGin(http.ServerConfig{
        Address:         cfg.HttpSrv.Address,
        ReadTimeout:     cfg.HttpSrv.ReadTimeout,
        WriteTimeout:    cfg.HttpSrv.WriteTimeout,
        BodyLimit:       8 << 20, // 8MB
        AccessLogFormat: http.DefaultAccessLogFormat,
        CORSConfig: &http.CORSConfig{
            AllowOrigins: []string{"*"},
            AllowMethods: []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
            AllowHeaders: []string{"Origin", "Content-Type", "Accept", "Authorization", "X-Request-ID"},
        },
    })

    routes.InitRoutes(router)

    srv := &http.Server{
        Addr:    cfg.HttpSrv.Address,
        Handler: router,
    }

    go func() {
        log.Infof("server starting on %s", cfg.HttpSrv.Address)
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("listen failed: %v", err)
        }
    }()

    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Infoln("shutting down server...")
    http.ShutdownServer(srv)
    repo.Close()
}
```

**说明：** 使用 `kit/http` 的 `NewGin()` 创建引擎，自动集成：
- `RequestIDMiddleware` — 请求ID生成和传递
- `AccessLogMiddleware` — 访问日志
- `CORS` — 跨域处理
- `ErrorHandler` — 统一错误处理

### 9. CLAUDE.md — 项目上下文

```markdown
# 项目上下文

## 1. 技术栈

- Go 1.25+
- Gin Web Framework
- MySQL + kit/sql
- JWT Authentication

## 2. 基础库

- `github.com/tiamxu/kit/sql` - 数据库封装
- `github.com/tiamxu/kit/log` - 日志封装
- `github.com/tiamxu/kit/http` - HTTP配置

## 3. 分层约束

| 层 | 职责 | 禁止 |
|---|---|---|
| `api/` | 参数解析、调用Service、返回响应 | 写业务逻辑、写SQL |
| `service/` | 业务逻辑、调用Repo | 写SQL、依赖HTTP细节 |
| `repo/` | 数据访问、SQL操作 | 写业务逻辑 |

## 4. 命名规范

- 文件：`小写蛇形.go`（如 `user_service.go`）
- 结构体：大写驼峰（如 `UserService`）
- 函数：`New` 前缀构造函数（如 `NewUserService`）

## 5. 错误处理

- 使用 `pkg/e` 统一错误码
- Service层错误包装：`fmt.Errorf("操作失败: %w", err)`
- 禁止原始error暴露给客户端

## 6. 开发命令

- 启动：`go run main.go`
- 测试：`go test ./...`
- 依赖：`go mod tidy`
```

### 10. README.md — 项目说明

```markdown
# Project Name

## 技术栈

- Go 1.23+
- Gin Web Framework
- MySQL + kit/sql
- JWT Authentication

## 项目结构

```
├── api/           # Handler层
├── service/       # Service层
├── repo/          # Repository层
├── model/         # 数据模型
├── types/         # DTO定义
├── middleware/    # 中间件
├── config/        # 配置加载
├── routes/        # 路由注册
├── pkg/e/         # 错误码
├── pkg/constants/ # 常量定义
├── pkg/utils/     # 工具函数
├── doc/           # 文档
└── sql/           # 数据库脚本
```

## 环境变量

```bash
export DB_USERNAME=root
export DB_PASSWORD=your_password
export ENV=dev
```

## 快速开始

```bash
go mod tidy
go run main.go
```

## API 规范

| 前缀 | 说明 | 鉴权 |
|---|---|---|
| `/public` | 公开接口 | 否 |
| `/api` | 用户接口 | JWT |
```

### 11. doc/设计文档.md — 设计文档模板

```markdown
# 功能设计文档

## 1. 概述

### 1.1 背景
描述需求背景和目标。

### 1.2 范围
明确功能范围和边界。

## 2. 技术方案

### 2.1 数据模型

| 字段 | 类型 | 说明 |
|---|---|---|
| id | bigint | 主键 |
| created_at | datetime | 创建时间 |

### 2.2 接口设计

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | /api/xxx | 创建 |
| GET | /api/xxx | 查询 |
| PUT | /api/xxx/:id | 更新 |
| DELETE | /api/xxx/:id | 删除 |

## 3. 实现计划

- [ ] 阶段一：数据库表设计
- [ ] 阶段二：Repo层实现
- [ ] 阶段三：Service层实现
- [ ] 阶段四：API层实现
```

### 12. doc/接口文档.md — 接口文档模板

```markdown
# 接口文档

## 统一响应结构

```json
{
    "code": 200,
    "message": "ok",
    "data": {}
}
```

## 错误码

| 错误码 | 说明 |
|---|---|
| 10001 | 参数错误 |
| 10002 | 未授权 |
| 20001 | 资源不存在 |
| 50000 | 服务器错误 |

---

## 接口列表

### POST /api/xxx

**请求参数：**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | string | 是 | 名称 |

**响应示例：**

```json
{
    "code": 200,
    "message": "ok",
    "data": {
        "id": 1
    }
}
```
```

---

## ⚡ 第三阶段：依赖安装

写完上述所有文件后，在当前目录的终端自动按顺序执行以下命令：

```bash
go mod init github.com/company/project
go get github.com/gin-gonic/gin
go get github.com/tiamxu/kit/sql
go get github.com/tiamxu/kit/log
go get github.com/golang-jwt/jwt/v5
go get github.com/koding/multiconfig
go mod tidy
```

确保所有文件编译通过，无任何错误。

---

## 📝 第四阶段：交付报告

初始化完成后，向用户输出：

```
✅ Go 项目脚手架创建完成！

📂 项目结构：
├── api/
│   └── common.go           # 统一响应
├── service/
├── repo/
│   └── init.go             # 数据库初始化
├── model/
├── types/
├── middleware/
│   └── auth.go             # JWT认证
├── config/
│   ├── config.go           # 配置加载
│   └── config-dev.yaml     # 开发环境配置
├── routes/
│   └── routes.go           # 路由注册
├── pkg/
│   ├── e/error.go          # 错误码
│   ├── constants/
│   └── utils/
├── doc/
│   ├── 设计文档.md
│   └── 接口文档.md
├── sql/
├── main.go                  # 程序入口
├── CLAUDE.md               # 项目上下文
└── README.md               # 项目说明

📋 依赖库：
├── github.com/gin-gonic/gin
├── github.com/tiamxu/kit/sql
├── github.com/tiamxu/kit/log
├── github.com/golang-jwt/jwt/v5
└── github.com/koding/multiconfig

📋 下一步：
1. 修改 config/config-dev.yaml 配置数据库连接
2. 设置环境变量 DB_USERNAME、DB_PASSWORD
3. 在 service/ 编写业务逻辑
4. 在 api/ 实现接口 Handler
5. 在 doc/ 编写设计文档和接口文档
```

## 约束

- 所有代码文件必须符合上述规范
- 不得跳过任何阶段
- go mod tidy 必须执行成功，无任何编译错误
- module名称用 `github.com/company/project` 替代
- 使用 `kit/sql`、`kit/log` 而非原生 sqlx