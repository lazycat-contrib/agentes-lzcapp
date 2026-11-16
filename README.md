# agentes-lzcapp

[Agentes](https://github.com/agentes-chat/agentes-chat) 的懒猫微服打包：开源、可自托管的团队聊天应用（类 Slack），人和 AI 智能体（Claude / ChatGPT(Codex) / Gemini / Grok）在频道与私信里协作，带 MCP 工具、例行任务、记忆与审批。AGPL-3.0。

- 上游镜像：`ghcr.io/agentes-chat/agentes`（GHCR 经 `ghcr.1ms.run` 加速），数据库 `pgvector/pgvector:0.8.7-pg18`（Docker Hub 经 `docker.1ms.run` 加速）
- 版本策略：镜像模式（`delivery.mode: mirror` + `require_digest_match`），跟随上游 SemVer tag 自动更新，只发布到喵喵商店
- `PUBLIC_URL=https://{{.S.AppDomain}}`：登录回跳、邀请链接与浏览器来源校验用应用自身域名
- 密钥：数据库口令与 `AGENTES_SECRET`（会话签名 + 已存模型 Key 的加密根密钥）由 `stable_secret` 模板生成，升级保持稳定
- `user: root`：上游镜像以 bun(1000) 运行，`/lzcapp/var` 由平台以 root 创建，root 启动保证数据目录可写
- 健康检查沿用镜像自带的 `bun -e fetch(.../api/health)`（容器内无 curl/wget 也不需要）
- 无部署期密码体系：首个注册账号即管理员，无需免密注入；注册在首个账号后自动关闭，成员靠邀请

## Agentes 的"电脑"（Agent computers）

智能体执行命令、驱动浏览器需要一台运行 runner 的"电脑"。应用容器本身不提供该能力（上游设计，
官方自托管也是独立接入）。用户在「Settings → Devices → Connect a device」复制安装命令，在
自己的电脑/服务器上运行即可接入；Claude Code / Codex 等订阅也是在那台电脑上登录。

## 说明

- 没配 SMTP 时，邀请链接/验证码写进应用日志（上游行为），可从日志复制。
- 语音转写、跨语言搜索等可选服务（speech / embeddings）未随包发布，需要时在上游文档基础上另行接入。

## 自动发布

`.github/workflows/lazycat.yml` 使用 [ca-x/lazycat-github-action](https://github.com/ca-x/lazycat-github-action)
`v1`：定时（每 3 小时）检查上游镜像新 tag，镜像模式校验 digest 后改写 manifest、构建 LPK、
发 GitHub Release（版本化 LPK 附件）并发布到喵喵商店。所需 Secrets（仓库级）：
`APPSTORE_URL`、`APPSTORE_TOKEN`。
