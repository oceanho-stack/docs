# ADR-002 公共 lib 拆分原则

- 状态：已采纳
- 日期：2026-11-02
- 决策人：Mimo（用户）& 架构协作者
- 关联项目：oceanho-stack / go-libs / services/iam-server

## 背景

开始 IAM 业务（tenant / user / team / role / permission）之前，用户明确要求**先抽离通用工具，后写业务**：
> "不应该是先 iam，应该是先抽离通用工具（日志、配置、db）"。

即：凡是不带业务语义、跨项目可复用的能力，先独立成库，业务项目通过依赖这些库组装，而不是在业务代码里重复实现或内嵌。

## 决策

### 1. 独立的 monorepo：go-libs

- 单仓库多库：`github.com/oceanho-stack/go-libs`。
- 每个库一个独立目录、独立 package，按需引入（业务只 import 自己用到的库）。
- go-libs 与业务项目**互相独立 git 仓库**。

### 2. 9 个公共库（全部完成）

| 库 | 职责 |
|---|---|
| xlog | 统一日志（slog 封装，链路字段透传） |
| xcfg | 配置加载（默认值→yaml→env→命令行） |
| xtrace | 链路追踪（context 透传 trace/span，header 注入） |
| xdb | GORM 底座（连接池/慢日志/事务/BasePageQuery 条件构造器） |
| xvalidator | 参数校验包装（中文错误翻译，自定义 id tag） |
| xid | ID 生成器（雪花算法，接口化 Generator） |
| xcrypto | 加密工具（bcrypt / AES-GCM / RSA / 随机 / 摘要） |
| xcache | 缓存抽象（Cache 接口 + Redis 实现 + 内存实现） |
| xhttp | HTTP 客户端（超时 / JSON / trace 注入 / 重试） |

### 3. 拆分判断标准（什么该进 lib）

一个能力**同时满足以下条件**才进 go-libs：
1. **无业务语义**：不依赖 tenant / user / role / permission / 业务错误码。
2. **跨项目可复用**：iam-server 之外（官网 / CLI / 其他服务）也会用，或测试/替换需要。
3. **只依赖标准库 + 稳定第三方库**（gorm、viper、go-redis 等）。

凡是带业务语义的（租户过滤、权限判定、业务 Resp 的 code 语义、AuthPrincipal 提取），**留在业务项目 pkg/ 或 domain/**，不进 lib。

### 4. 硬性依赖规则

- **禁止反向依赖业务项目**（go-libs 绝不 import iam-server）。
- lib 之间按依赖方向组织：`xlog → xcfg → xtrace → xdb → xvalidator/xid/xcrypto → xcache → xhttp`。
- 接口优先：对外暴露接口（`Generator`、`Cache`），便于测试用内存/替身实现替换，也便于未来换底层（如换日志后端、缓存后端）。

### 5. 交付规范（每个库）

1. 先写 README（用途 + 快速开始 + 设计边界）。
2. 实现 + 单元测试。
3. `go build` / `go vet` / `go test` 全过。
4. commit + push 到 go-libs 远程。
5. 同步更新 docs 仓库清单与状态。

## 后果

优点：
- 业务代码干净，公共能力一次写好到处用。
- 每个库独立测试，无业务干扰。
- 未来新增端（官网 / CLI / App）直接复用。

代价 / 取舍：
- 库越多，维护面越宽；需克制"什么都想抽"的冲动，按判断标准走。

## 关联 ADR
- ADR-001 分层与仓储查询封装规范
- ADR-003 统一请求响应协议
- ADR-004 GORM 使用规范（数据库无关）
