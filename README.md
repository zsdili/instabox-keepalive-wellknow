# InstaBox 免费云主机保活

用双保险机制让 `wellknow-box`（4核8G 免费机器）始终保持 `running`，不被 30 分钟空闲挂起。

## 保活架构（双保险）

### 1. 机器内部保活（主，已验证稳定）⭐
- `systemd` 服务 `instabox-selfkeep`：每 5 分钟调官方 activity lease 接口续租，开机自启、崩溃自动重启。
- 脚本：`/data/gobin/scripts/self-keepalive.sh`（项目副本 `instabox-go-starter/scripts/self-keepalive.sh`）
- 日志：`/data/gobin/logs/self-keepalive.log`
- 原理：租约让机器不休眠 → 机器 running 时该循环才能运行 → 继续续租，形成自维持。

### 2. GitHub Actions（辅，手动可靠）
- 本仓库 workflow `keepalive.yml`，手动触发 **Run workflow** 可随时续租。
- 注意：GitHub 对新仓库/低活跃仓库的 scheduled cron 有抑制，可能不自动触发，故不作为唯一依赖。

## 原理
- InstaBox 免费机器空闲 30 分钟自动 suspend，账号级锁死无法关闭。
- 官方 activity lease 接口 `PUT /v1/machines/{id}/activity-leases/{name}` 可续租阻止挂起，鉴权头 `X-API-Key`。
- API 列机器返回 `{"machines":[...]}`。

## 运维命令（在机器上）
```bash
systemctl status instabox-selfkeep          # 内部保活状态
sudo systemctl restart instabox-selfkeep    # 重启
tail -f /data/gobin/logs/self-keepalive.log # 续租日志
curl -s localhost:8080/healthz              # 服务健康
```

## 安全
- API key 仅 `machines:write` 权限，存于 GitHub Secrets 与机器脚本，别外泄。
