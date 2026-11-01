# ADR-004 GORM 使用规范（数据库无关）

- 状态：已采纳
- 日期：2026-11-02
- 决策人：Mimo（用户）& 架构协作者
- 关联项目：oceanho-stack / services/iam-server

## 背景

数据库操作统一用 GORM v2，但**不强制绑定数据库特性**。本机环境无 cgo（CGO_ENABLED=0），且希望代码能在 PostgreSQL / MySQL / SQLite 等不同库间迁移，因此所有 GORM 用法必须数据库无关。

## 决策

### 1. 数据库无关（硬性约束）

- **禁止写数据库专属 SQL 方言**（如 MySQL 的 `ON DUPLICATE KEY`、PostgreSQL 的 `RETURNING` 非标准用法）。
- **禁止使用数据库专属类型**（如 PostgreSQL 的 `jsonb` 数组专属操作、MySQL 的 `ENUM`）。
- 尽量用 GORM 的标准方法（`Create` / `First` / `Find` / `Where` / `Preload` / `Transaction` / `AutoMigrate`），跨库行为一致的部分。
- 分页、排序、过滤一律走仓库层的 Query 构造器，不散写 SQL。

### 2. 仓库层统一入口

- 所有数据库操作只发生在 `repository` 层。
- service / handler **禁止直接调用 `db.Where(...)`**。
- 每个实体一个 `xxxQuery`（条件构造器，链式返回副本）+ `xxxRepo`（List/Get/Create/Update/Delete）。
- 基础查询能力来自 xdb：
  - `BasePageQuery`：分页、排序、预加载，通用。
  - `TenantScopedQuery`：内嵌 BasePageQuery，附加 tenantID 租户过滤（仅租户隔离资源使用）。
- `*gorm.DB` 句柄不对外暴露；repo 通过 xdb 的 DB() 获取并 `WithContext(ctx)`。

### 3. 事务

- 涉及多写操作一律用事务：`xdb.Tx(ctx, func(tx *gorm.DB) error { ... })`。
- 业务事务编排在 service 层，repo 只提供单条目的增删改查。

### 4. 测试

- 单元测试用**内存数据库**：`github.com/glebarez/sqlite`（纯 Go，无 cgo）。
- 每个测试用**唯一内存 DSN**，避免跨测试共享 `:memory:` 累积数据：
  ```
  file:<TestName>_<unixnano>?mode=memory&cache=shared
  ```
- 不依赖本机安装 PostgreSQL / MySQL / Redis 即可跑 xdb 测试（redis 用内存实现替身测接口契约）。

### 5. 踩坑记录（勿重试）

- `gorm.io/driver/sqlite`：需要 cgo，本机 CGO_ENABLED=0 不可用 → 改用 `github.com/glebarez/sqlite`。
- PostgreSQL 驱动在 `gorm.Open` 即使 DryRun 也会尝试真实连接 → 测试统一用内存 sqlite。
- 共享 `file::memory:` DSN 会导致测试间数据累积 → 每个测试唯一 DSN。
- Windows 无管理员权限无法 winget/MSI 装 Go → 用阿里云镜像下载 zip 解压 + 用户级环境变量（详见「环境搭建」文档）。

## 后果

优点：
- 代码可在 PostgreSQL / MySQL / SQLite 间切换，便于本地轻量开发与生产部署。
- 测试无需真实数据库，CI 可跑。

代价 / 取舍：
- 放弃部分数据库高性能特性（需在业务明确需要时单独评估，加注释说明）。

## 关联 ADR
- ADR-001 分层与仓储查询封装规范
- ADR-002 公共 lib 拆分原则
- ADR-003 统一请求响应协议
