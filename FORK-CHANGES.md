# FORK-CHANGES

本仓库 fork 自 [QuantumNous/new-api](https://github.com/QuantumNous/new-api)，本文件用于记录与上游的所有自定义差异，方便后续同步与维护。

> 上游官方仓库：<https://github.com/QuantumNous/new-api>
> 本仓库：<https://github.com/xjxnx/new-api>

---

## 当前同步进度

| 项目 | 信息 |
| --- | --- |
| 最近一次同步上游版本 | `v1.0.0-rc.2`（2026-04-30 发布） |
| 同步方式 | `git merge upstream/main` |
| 同步合并提交 | `d2ac461c Merge remote-tracking branch 'upstream/main'` |

---

## 自定义改动清单

| # | 类别 | 涉及文件 | 说明 |
| --- | --- | --- | --- |
| 1 | Logo 替换 | `web/default/public/logo.png` | 新版前端 logo |
| 2 | Logo 替换 | `web/classic/public/logo.png` | 经典前端 logo |
| 3 | 在线客服 | `web/default/index.html` | 嵌入 Tawk.to 客服脚本 |
| 4 | 在线客服 | `web/classic/index.html` | 嵌入 Tawk.to 客服脚本 |
| 5 | CI/CD | `.github/workflows/ghcr-publish.yml` | 自动构建并推送镜像到 GHCR |

### 改动 1 & 2：Logo 替换

将原作者的默认 logo 替换为自定义的 M 字形 logo 图片。

- 上游路径变化：v1.0 之前是 `web/public/logo.png`，v1.0 之后拆分为 `web/default/public/logo.png` 和 `web/classic/public/logo.png`，**两份都需要替换**。
- 文件大小：约 11.5 KB（PNG）。
- 前端引用路径：`/logo.png`（在 `web/{default,classic}/index.html` 的 `<link rel="icon">` 中）。
- Logo 兜底逻辑见 `web/{default,classic}/src/helpers/utils.jsx` 中的 `getLogo()` 函数。

### 改动 3 & 4：Tawk.to 在线客服

在前端 `<body>` 末尾注入 Tawk.to 客服 JS。

```html
<script type="text/javascript">
  var Tawk_API = Tawk_API || {}, Tawk_LoadStart = new Date();
  (function() {
    var s1 = document.createElement("script"),
        s0 = document.getElementsByTagName("script")[0];
    s1.async = true;
    s1.defer = true;
    s1.src = 'https://embed.tawk.to/6979bdb9ea4d47197d0e1f80/1jg1oq7pa';
    s1.charset = 'UTF-8';
    s1.setAttribute('crossorigin', '*');
    s0.parentNode.insertBefore(s1, s0);
  })();
</script>
```

> 客服站点 ID：`6979bdb9ea4d47197d0e1f80`
> 小部件 ID：`1jg1oq7pa`

### 改动 5：GHCR 自动构建 Workflow

新增 `.github/workflows/ghcr-publish.yml`，每次 `push` 到 `main` 分支或手动触发时，会自动：

1. 构建 `linux/amd64` + `linux/arm64` 双架构镜像
2. 推送到 `ghcr.io/xjxnx/new-api`
3. 创建多架构 manifest，打上 `latest` 和 `custom-YYYYMMDD-<sha>` 两个 tag

**未修改原作者已有的 workflow**（`docker-build.yml` / `docker-image-nightly.yml` / `docker-image-alpha.yml` 等保持原样）。

---

## 同步上游的步骤

每次想从原作者同步新功能时，按以下顺序操作：

```powershell
# 1. 确认 upstream 已配置（一次性，已配好）
git remote -v
# 应包含：upstream  https://github.com/QuantumNous/new-api.git (fetch/push)

# 2. 备份你的自定义文件（可选但推荐）
copy web\default\public\logo.png logo.png.bak
copy web\classic\public\logo.png logo.classic.png.bak
copy web\default\index.html default.index.html.bak
copy web\classic\index.html classic.index.html.bak

# 3. 拉取并合并上游
git fetch upstream
git merge upstream/main

# 4. 处理冲突（重点检查这些文件）
#    - web/default/index.html      → 保留 Tawk.to 脚本
#    - web/classic/index.html      → 保留 Tawk.to 脚本
#    - web/default/public/logo.png → 用你的 logo（必要时从备份恢复）
#    - web/classic/public/logo.png → 用你的 logo（必要时从备份恢复）

# 5. 提交合并
git add .
git commit -m "sync: merge upstream <版本号>"

# 6. 推送
git push origin main
```

GitHub Actions 会自动构建新镜像，等构建完成后服务器侧执行：

```bash
docker compose pull
docker compose up -d
```

---

## 部署信息

### Docker 镜像

- 镜像名：`ghcr.io/xjxnx/new-api:latest`
- 仓库：<https://github.com/users/xjxnx/packages/container/new-api>
- 可见性：**Public**（首次构建后需手动改为 Public）

### 服务器 docker-compose 关键配置

```yaml
services:
  new-api:
    image: ghcr.io/xjxnx/new-api:latest
    container_name: new-api
    restart: always
    # ...其余保持上游默认即可
```

### 服务器更新流程

```bash
docker compose pull
docker compose up -d
```

---

## 维护提醒

### 不能被上游覆盖的文件

合并上游时务必确保这些文件保留你的版本：

- `web/default/public/logo.png`
- `web/classic/public/logo.png`
- `web/default/index.html`（Tawk.to 脚本不能丢）
- `web/classic/index.html`（Tawk.to 脚本不能丢）
- `.github/workflows/ghcr-publish.yml`（不要被上游误删）
- `FORK-CHANGES.md`（本文件）

### 如果上游再次重构前端目录

历史教训：v1.0 把 `web/index.html` 拆成了 `web/default/` 和 `web/classic/` 两份。如果未来上游又有类似重构（例如新增 `web/mobile/` 之类），需要：

1. 把 logo 同步到所有新目录的 `public/logo.png`
2. 把 Tawk.to 脚本注入到所有新目录的 `index.html`
3. 更新本文件「自定义改动清单」表格

### 如果不再需要某项改动

直接删除对应文件/代码段，并更新本文件清单。

---

## 变更历史

| 日期 | 改动 |
| --- | --- |
| 2026-05-06 | 初始化本文件，记录 logo / Tawk.to / GHCR workflow 三项改动 |
| 2026-05-06 | 同步上游 `v1.0.0-rc.2` |
