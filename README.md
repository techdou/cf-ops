# cf (Cloudflare CLI)

Cloudflare 官方新一代 CLI「cf」（wrangler 继任者）的使用技能：cf 镜像整个 Cloudflare API（3000+ 操作），覆盖 DNS 记录 / 域名 zone 管理、Workers 部署、WAF、Access、R2 / D1 / KV 与账号资源。凡用 `cf` 命令操作 Cloudflare 平台的任务先加载本技能。入口与完整规则见 [SKILL.md](SKILL.md)。

注意与 `cloudflared`（隧道 daemon）无关——内网穿透任务用 cloudflared，不用 cf。

## 安装与更新

仓库根目录就是标准技能目录；选择宿主支持的 skills 路径安装一次。

```bash
# 多宿主共享路径
git clone https://github.com/techdou/cf.git ~/.agents/skills/cf

# 或安装到 Claude Code 路径
git clone https://github.com/techdou/cf.git ~/.claude/skills/cf
```

若目录已是 Git 检出，用 `git pull --ff-only` 更新；出现分叉先合并，不覆盖本地改动。

## 使用

核心安全流程（每条不熟悉的命令都走一遍）：

```bash
cf cli search "任务描述"   # 找命令，免登录，返回 JSON（最多 5 条）
cf schema <command>        # 查该命令的完整参数 schema
cf <command> --dry-run     # 预览将执行的 API 调用，不动真资源
cf <command>               # 确认后执行
```

凭证由 cf 自身管理（`cf auth login` 浏览器 OAuth / `CLOUDFLARE_API_TOKEN` 环境变量做 CI），本技能不持有任何凭证。

收录的官方坑点：

1. **静默中止**：非交互会话中破坏性命令不加 `--force` 会打印 `Aborted.` 且退出码为 0——exit 0 ≠ 已执行，必须检查输出文本。
2. **--force 双义**：某些命令的 `--force` 本身是 API 参数而非确认开关，授权给 agent 时要谨慎。
3. **wrangler 项目别乱动**：无 `cloudflare.config.ts` 时先 `cf migrate --dry-run` 预览，存量项目不必强迁。

## 与兄弟工具的分工

| 工具 | 角色 | 何时用 |
|------|------|--------|
| `cf` | 平台管理 CLI（wrangler 继任者） | DNS、域名 zone、账号资源、Workers 部署、WAF 等平台操作 |
| `wrangler` | 上一代 Workers CLI | 存量 wrangler 项目 |
| `cloudflared` | Tunnel 隧道 daemon | 内网穿透、把本地服务暴露到公网 |

## 维护与来源

cf 处于 beta 期，命令结构可能变化；本技能永远以 `cf cli search` / `cf schema` / 官方文档实测为准，不依赖预训练记忆。MIT 许可见 [LICENSE](LICENSE)。
