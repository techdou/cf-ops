---
name: cf
description: Cloudflare 官方新一代 CLI（wrangler 继任者），镜像整个 Cloudflare API（3000+ 操作）：DNS 记录/域名 zone 管理、Workers 部署、WAF、Access、R2/D1/KV、账号资源等。凡是用 cf 命令操作 Cloudflare 平台的任务先加载本 skill。注意与 cloudflared（隧道 daemon）无关——内网穿透任务用 cloudflared，不用 cf。
---

# cf CLI (Cloudflare)

cf 是 Cloudflare 的 agentic CLI，v1.0.0-beta 起替代 wrangler 的定位。**beta 期命令结构可能变化，永远以下方检索流程实测为准，不靠预训练记忆。**

## 凭证说明

本 skill 无需 .env。认证由 cf 自身管理（凭据存 `~/.config/cloudflare`）：

- 交互登录：`cf auth login`（浏览器 OAuth，只能由豆哥本人执行，agent 代劳不了）
- 查状态：`cf auth whoami`
- 无人值守/CI：设环境变量 `CLOUDFLARE_API_TOKEN`（优先级高于已存登录信息）
- 多项目隔离：`cf auth create <name>` 建 named profile + `cf auth activate <name> [dir]` 绑定目录

## 核心安全流程（每条不熟悉的命令都走）

```
cf cli search "任务描述"   # 找命令，免登录，返回 JSON（最多 5 条）
cf schema <command>        # 查该命令的完整参数 schema
cf <command> --dry-run     # 预览将执行的 API 调用，不动真资源
cf <command>               # 确认后执行
```

## 输出与过滤

- 输出默认 JSON，直接接 `jq` 过滤，不要加多余的解析层
- `--quiet` 抑制非必要输出
- zone 参数可直接给域名：`cf dns records list -z example.com`（不用先查 zone id）

## 坑点（官方文档明示）

1. **静默中止**：非交互会话中，破坏性命令不加 `--force` 会打印 `Aborted.` 且**退出码为 0**——exit 0 ≠ 已删除，必须检查输出文本。
2. **--force 双义**：某些命令的 `--force` 本身是 API 参数而非确认开关（如 `cf workers delete --force`），授权给 agent 时要谨慎。
3. **wrangler 项目别乱动**：项目里有 wrangler 配置但无 `cloudflare.config.ts` 时，不要跑 `cf dev/build/deploy`——先 `cf migrate --dry-run` 预览迁移。
4. wrangler 迁移后仍有长期维护窗口，存量项目继续用 wrangler 即可，不必强迁。

## 与兄弟工具的分工

| 工具 | 角色 | 何时用 |
|------|------|--------|
| `cf` | 平台管理 CLI（wrangler 继任者） | DNS、域名 zone、账号资源、Workers 部署、WAF 等一切平台操作 |
| `wrangler` | 上一代 Workers CLI | 存量 wrangler 项目 |
| `cloudflared` | Tunnel 隧道 daemon | 内网穿透、把本地服务暴露到公网；与 cf 无关 |

## 常用命令速查（以 search/schema 实测为准）

```bash
cf auth whoami                          # 登录状态
cf zones list                           # 列出所有域名
cf dns records list -z example.com      # 列 DNS 记录
cf dns records create -z example.com --help   # 建记录前先看 schema
cf tunnel --help                        # tunnel 相关（cloudflared 的云端配置部分）
cf cli                                  # CLI 自身的发现/配置命令组
```

## 检索来源（冲突时以此为准，不信本文件）

- `cf cli search` + `cf schema` + `--help`（一手、随版本更新）
- https://developers.cloudflare.com/cf/ （官方文档）
- cloudflare-docs MCP search 工具（本环境已配）
