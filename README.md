# InstaBox 免费云主机保活

用 GitHub Actions 的定时任务，每 5 分钟调用一次 InstaBox 官方 activity lease 接口续租，让免费机器（4 核 8G）保持 running，不被 30 分钟空闲自动挂起。

## 原理

- InstaBox 免费机器空闲 30 分钟会自动 suspend，且不能关闭（`fixedAutoSuspendSeconds=1800`）。
- 官方提供 **activity lease** 接口：对机器续租一个租约（`PUT /v1/machines/{id}/activity-leases/{name}`），即可阻止空闲挂起。
- 租约必须来自机器外部（机器 suspend 后内部定时任务会冻结），因此用 GitHub Actions 的 cron 定时调用，稳定且免费（公开仓库 Actions 分钟数不限）。
- 机器若已被挂起，脚本会先 `resume` 再续租。

## 配置

- `INSTABOX_API_KEY`（Repository Secret）：InstaBox 账号 API key（`ibk_...`），权限 `machines:write` 即可。
- 定时：`cron: '*/5 * * * *'`，租约 TTL 900 秒（有 3 倍余量）。
- 可到 Actions 页面手动点 **Run workflow** 立即自检。

## 安全

- 只使用最小权限（`machines:write`）的 key，不含删除/镜像权限。
- key 只存 GitHub Secrets，不进代码。
