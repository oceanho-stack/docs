# oceanho-stack（docs 知识库）

> 这是 `oceanho-stack` 工作空间的**文档仓库**，也是一份**供人读、供 AI 读的知识库**。
> 无论是你手动开发，还是用 vibe coding（AI 结对编码），都先读本文件定位到对应文档。

## 一句话

自研一套**多端共用**的 Go 多租户身份与租户底座（IAM）：产品官网、Console、手机 App、CLI 共用同一套 API 与协议。后端 Go（Gin + GORM v2 + Casbin + Redis + PostgreSQL），前端 Vue3 + Vben Admin（Console）/ Nuxt3（官网）。

## 知识库入口（读我导航）

| 想解决什么 | 看这里 |
|---|---|
| 整个工作空间的目录/仓库怎么组织的 | [目录结构建议.md](目录结构建议.md) |
| 分层与仓储查询封装规范 | [adr/ADR-001-分层与仓储查询封装规范.md](adr/ADR-001-分层与仓储查询封装规范.md) |
| 公共 lib 怎么拆、go-libs 里有哪些库 | [adr/ADR-002-公共lib拆分原则.md](adr/ADR-002-公共lib拆分原则.md) |
| 统一请求/响应协议、错误码、Header | [adr/ADR-003-统一请求响应协议.md](adr/ADR-003-统一请求响应协议.md) |
| GORM 数据库无关规范、测试、踩坑 | [adr/ADR-004-GORM使用规范.md](adr/ADR-004-GORM使用规范.md) |
| 编码/命名/提交规范（写代码前必读） | [standards/编码与规范.md](standards/编码与规范.md) |
| 新电脑 / 新环境怎么还原 | [目录结构建议.md](目录结构建议.md)（第三节） |

> `api/`、`db/`、`notes/` 目录按需生长：有了接口定义、表结构、草稿再创建，不预建空目录。

## 当前进度（2026-11-02）

- ✅ **go-libs 9 个公共库全部完成**并推送 GitHub（见 ADR-002 表格）。
- ⬜ iam-server 未创建（下一步：按「先工具后业务」顺序，工具已齐，可开业务）。

## 已确立的技术决策速览

- **先抽工具后业务**：通用能力先进 go-libs，业务项目只组装（ADR-002）。
- **分层**：domain / dto / repository / service / handler / policy + pkg（ADR-001）。
- **查询封装**：Query 链式返回副本，repo 不写业务，handler 薄（ADR-001）。
- **统一协议**：`Resp[T]`、`code` 分段、分页、Header 约定（ADR-003）。
- **GORM 数据库无关**：不写方言/专属类型，测试用内存 sqlite（ADR-004）。

## 仓库清单

| 仓库 | 远程 | 作用 |
|---|---|---|
| oceanho-stack/docs | https://github.com/oceanho-stack/docs.git | 本知识库 |
| oceanho-stack/go-libs | https://github.com/oceanho-stack/go-libs.git | 公共 Go 工具库（9 库完成） |
| oceanho-stack/iam-server | （待创建） | IAM 多租户服务（主服务） |

## 给 vibe coding / AI 的提示

1. **动手前先读**：改代码前先读本 README 定位相关 ADR 与规范，尤其 `standards/编码与规范.md`。
2. **遵守边界**：`domain` 不 import gorm/gin/casbin；`db.Where` 只在 repository；通用能力先进 lib。
3. **遵循统一协议**：新接口返回 `Resp[T]`，错误码按 ADR-003 分段。
4. **测试必带**：新增/修改功能补单元测试，`go build` / `go vet` / `go test` 全过后再提交。
5. **Go 环境**：本机 Go 在 `D:\dev\go`，新 shell 需设置 PATH/GOROOT/GOPATH（见「环境搭建」备注，或目录结构建议末尾）。
