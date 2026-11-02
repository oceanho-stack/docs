# 开源 IAM / OAuth Provider 对比（2026-11）

> 目的：评估"是否复用成熟开源组件，避免造轮子"。对比维度：定位 / 语言 / 许可证 / 成熟度(star) / 能力 / 多租户 / **形态（能否嵌入）**。

## 一、完整 IAM 服务（自托管独立服务）

| 项目 | 语言 | 许可证 | star | 定位 | OIDC/OAuth2 | SAML | 多租户 | 形态 |
|---|---|---|---|---|---|---|---|---|
| **Keycloak** | Java (Quarkus) | Apache-2.0 | ~22k+ | 企业级完整 IAM | ✅ | ✅ | ✅ (realm) | **独立服务** |
| **Ory (Kratos+Hydra+Keto)** | Go | Apache-2.0 | Hydra ~16k / Kratos ~11k | 模块化云原生 IAM | ✅ | ✅ | 无(每实例一租户) | **独立服务/库(可嵌入)** |
| **ZITADEL** | Go | Apache-2.0(→AGPL v3) | ~13k | B2B 多租户云原生 IAM | ✅ | ✅ | ✅ (org/instance) | 独立服务 |
| **Authentik** | Python/Django | MIT(核心) | ~17k | 灵活 IAM+代理 | ✅ | ✅ | ✅ | 独立服务 |
| **Casdoor** | Go | Apache-2.0 | ~13.8k | 界面友好 IAM | ✅ | ✅ | ✅ | 独立服务 |
| **Logto** | JS/TS | Apache-2.0 | ~8k | SaaS/App 开发者优先 IAM | ✅ | 部分 | ✅ | 独立服务 |
| **SuperTokens** | Java(核心)/JS | Apache-2.0 | ~15k | Auth0 替代 | ✅ | 无 | 部分 | 独立服务+SaaS |
| **Authelia** | Go | Apache-2.0 | ~24k | 轻量 2FA/SSO 反向代理 | ✅ | 无 | 无 | 反向代理形态 |
| **Dex** | Go | Apache-2.0 | ~11k | k8s/云原生 OIDC 连接 | ✅ | 无 | 无 | 轻量 OIDC 服务器 |

## 二、Go 可嵌入的库（符合我们"库+壳"形态）

| 库 | 语言 | 许可证 | 能力 | 能否嵌入 | 说明 |
|---|---|---|---|---|---|
| **Ory Fosite** | Go | Apache-2.0 | 完整 OAuth2 + OIDC 服务器逻辑 | ✅ | **可嵌入的 Go 库**，Hydra 的内核，广泛生产使用 |
| **ory/fosite** | Go | Apache-2.0 | OAuth2/OIDC 授权码/凭证/刷新 | ✅ | 认证协议层，业界最成熟的 Go 实现 |
| **ory/hydra 嵌入模式** | Go | Apache-2.0 | OIDC Provider | ✅ | 文档明确支持 "embed Hydra as a Go library" |
| **go-oidc (hubsystem)** | Go | MIT | OIDC Provider/客户端 | ✅ | OpenID Foundation 认证过，认证服务器 |
| **golang-jwt/jwt** | Go | MIT | JWT 签发/校验 | ✅ | 配合 kid+JWKS 轮换 |
| **ory/keto** | Go | Apache-2.0 | 授权（Zanzibar 风格） | ✅ | 与 Casbin 竞争，可评估 |

## 三、关键结论（对 iam-core 的决策意义）

### 1. 现有开源 IAM 都是"独立服务"，没有"可 SDK 嵌入的多租户库"
Keycloak/ZITADEL/Casdoor/Logto 都是**独立部署的服务**（JVM/Python/独立进程），不是可被 A/B/C 直接 `import` 的 Go 库。这与我们"库+壳双形态、不共享场景各自嵌入"的定位**不匹配**。

### 2. 我们真正需要复用的是"认证协议层"，不是整个 IAM 服务
我们的价值在**多租户隔离 + 身份领域建模 + 业务权限**（这些现成开源没有、且不该外包）。而**认证协议层（OAuth2/OIDC 流程、JWT、kid/JWKS 轮换）是成熟可复用的**，**应该复用 Ory Fosite / golang-jwt**，避免重写协议细节。

### 3. 建议的组合（避免造轮子 + 保留差异化）
```
iam-core
├── 认证协议层：复用 Ory Fosite（OAuth2/OIDC 服务器逻辑）+ golang-jwt（JWT/kid/JWKS）
├── 身份/租户领域：自研（Tenant/User/Membership/Team，开源无此通用库）
├── 授权：复用 Casbin（已定）或评估 Ory Keto
└── 壳 iam-server：复用 Fosite 内核包 HTTP API
```

### 4. 何时才该考虑直接用现成完整 IAM
- 如果**不需要"库形态嵌入"**，愿意接受"每个项目都调一个独立 IAM 服务"，那么直接上 **Keycloak 或 ZITADEL** 最省事。
- 但我们已定"库+壳、不共享各自嵌入" → 现成服务不匹配，**自研内核 + 复用协议库**是正确的。

---

## 参考来源
- Ory vs Keycloak: https://www.ory.com/comparisons/ory-vs-keycloak
- Casdoor 特性对比: https://casdoor.ai/uk/docs/comparison/feature-comparison/
- 开源 IAM 列表: https://selfhosting.sh/best/authentication-sso/
- ZITADEL 官网: https://zitadel.com/
- Ory 开源组件: https://www.ory.com/open-source
- OAuth Go 库列表: https://oauth.net/code/go/
- Ory Fosite (可嵌入): https://www.ory.com/community
- GitHub star: casdoor 13.8k / zitadel 13k / supertokens 14.9k / ory-hydra 16k
