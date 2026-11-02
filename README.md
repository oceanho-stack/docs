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
| 开发前先调研开源成熟方案（避免造轮子） | [adr/ADR-005-开发前先调研开源成熟方案.md](adr/ADR-005-开发前先调研开源成熟方案.md) |
| 开源 IAM / OAuth Provider 调研对比 | [调研/开源IAM与OAuth对比.md](调研/开源IAM与OAuth对比.md) |
| 编码/命名/提交规范（写代码前必读） | [standards/编码与规范.md](standards/编码与规范.md) |
| 新电脑 / 新环境怎么还原 | [目录结构建议.md](目录结构建议.md)（第三节） |

> `api/`、`db/`、`notes/`、`projects/<repo>/` 目录按需生长：有了接口定义、表结构、草稿再创建，不预建空目录。

### 知识库规则（RAG 友好，AI 与人都适用）

1. **README 是唯一入口**：AI 或人先读本 README，通过上方导航表定位到具体文档，不逐文件翻找。
2. **新文档必须登记**：任何新增文档（ADR / 规范 / 调研 / 项目文档），**必须在导航表登记一行**，否则视为未归档。
3. **按需生长**：目录只在有真实内容时创建，不预建空目录，避免检索噪音。
4. **分类清晰**：`adr/`=决策记录，`standards/`=编码规范，根文档=环境与组织，`projects/`=各仓库专属文档。

## 当前进度（2026-11-02）

- ✅ **go-libs 9 个公共库全部完成**并推送 GitHub（见 ADR-002 表格）。
- ✅ **iam-core 已创建**：IAM 设计文档（DESIGN.md）+ domain 骨架在推进中（见 `projects/iam-core/`）。
- ⬜ iam-server 未创建（工具已齐，认证协议层拟复用 Ory Fosite，见 ADR-005）。

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
| oceanho-stack/iam-core | https://github.com/oceanho-stack/iam-core.git | IAM 多租户内核（DESIGN + domain） |
| oceanho-stack/iam-server | （待创建，iam-core 同仓库子目录） | IAM 服务壳（共享场景权威，认证协议复用 Ory Fosite） |

## 给 vibe coding / AI 的提示

1. **动手前先读**：改代码前先读本 README 定位相关 ADR 与规范，尤其 `standards/编码与规范.md`。
2. **遵守边界**：`domain` 不 import gorm/gin/casbin；`db.Where` 只在 repository；通用能力先进 lib。
3. **遵循统一协议**：新接口返回 `Resp[T]`，错误码按 ADR-003 分段。
4. **测试必带**：新增/修改功能补单元测试，`go build` / `go vet` / `go test` 全过后再提交。
5. **Go 环境**：本机 Go 在 `D:\dev\go`，新 shell 需设置 PATH/GOROOT/GOPATH（见「环境搭建」备注，或目录结构建议末尾）。
