# InstaBox 免费云主机保活

用 GitHub Actions 定时任务，每 5 分钟调用 InstaBox 官方 activity lease 接口续租，让免费机器（4 核 8G）保持 `running`，不被 30 分钟空闲自动挂起。

## 当前状态（已部署并验证通过）

| 项目 | 值 |
|------|-----|
| InstaBox 账号 | zsdili（免费版）|
| 机器 | wellknow-box（4 vCPU / 8 GB / 20 GB 持久盘）|
| 机器 ID | m_agq7uyx5ft27oh4wi2ki4rdpee |
| GitHub 仓库 | https://github.com/zsdili/instabox-keepalive-wellknow |
| 定时任务 | `.github/workflows/keepalive.yml`，`cron: */5 * * * *` |
| API key | 仅 `machines:write` 最小权限，存于 GitHub Secrets `INSTABOX_API_KEY` |
| 验证结果 | 手动触发运行成功，`lease HTTP 200`，租约续到 +15 分钟 |

## 原理

- InstaBox 免费机器空闲 30 分钟自动 suspend，且账号级锁死无法关闭（`fixedAutoSuspendSeconds=1800`）。
- 官方 activity lease 接口 `PUT /v1/machines/{id}/activity-leases/{name}` 可以续租阻止挂起。鉴权头用 `X-API-Key`。
- 租约必须来自机器外部（机器 suspend 后内部定时全冻结），所以用 GitHub Actions cron 定时调用，公开仓库 Actions 分钟数不限、完全免费。
- 机器若已被挂起，脚本会先 `POST /resume` 再续租。
- API 列机器返回结构是 `{"machines":[...]}`，脚本用 `.machines[]` 提取。

## 日常使用

- 查看保活运行记录：打开仓库的 **Actions** 标签页。
- 立即自检：Actions → InstaBox Keepalive → **Run workflow**。
- 改续租频率：编辑 `keepalive.yml` 里的 `cron`（TTL 900 秒，建议间隔不超过 5 分钟）。
- 连接机器：`npx instabox@latest sh` 或 `instabox ssh wellknow-box`。

## 安全

- 只使用 `machines:write` 权限的 key，不含删除/镜像权限；旧的高权限 key 已吊销。
- key 只存 GitHub Secrets，不进代码。如需轮换：在 InstaBox 控制台 Profile → API keys 重建，再更新仓库 Secret。
