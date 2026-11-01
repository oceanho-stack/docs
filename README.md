# oceanho-stack（文档仓库）

本仓库是 `oceanho-stack` 工作空间的**文档仓库**，独立 git 管理。

> 工作空间根目录：`D:\ocean\project\oceanho-stack`
> 说明：`oceanho-stack` 本身不是 git 仓库，是本地收纳盒；文档（本仓库）与各项目各自独立 git。

## 仓库结构

```
docs/                       ← 本仓库（oceanho-stack/docs）
├── README.md               # 本文档
├── 目录结构建议.md          # 新电脑还原结构的说明书（重要）
├── adr/                    # 架构决策记录 ADR
├── api/                    # OpenAPI、接口样例、错误码定义
├── db/                     # 表结构、ER 图、迁移说明
├── standards/              # 编码规范、仓库约束、命名规范
└── notes/                  # 临时沟通草稿、想法记录
```

> 结构按需生长：目录随实际文档产生而创建，不预建空目录。
> 新电脑还原请参照 `目录结构建议.md`。

## 已确立的架构决策（详见 adr/）

| 编号 | 主题 | 状态 |
|---|---|---|
| ADR-001 | 分层与仓储查询封装规范 | 已采纳 |

## 关键技术约定（速览）

- 后端：Go（Gin + GORM v2 + Casbin + Redis + PostgreSQL）
- 前端：Vue3 + Vben Admin（Console）；Nuxt3（官网）；后续可另起 React 对接同一套 API
- ORM：GORM，不强制绑定数据库特性（不写方言 SQL / 数据库专属类型）
- 协议：统一 `Resp[T]` / `PageReq` / `PageResp`，多端共用
- 领域模型：User 全局、可加入多租户；租户内树形团队；租户/团队两级角色
