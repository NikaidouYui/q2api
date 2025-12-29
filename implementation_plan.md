# 自动同步与 Docker 构建实现计划

## 目标
1.  **自动同步上游**: 定期从上游仓库 (`CassiopeiaCode/q2api`) 拉取更新并合并到当前 Fork 仓库。
2.  **自动构建镜像**: 当代码更新（包括同步产生的更新）时，自动构建 Docker 镜像并推送到 GitHub Container Registry (GHCR)。

## 实现方案

### 1. 同步工作流 (`.github/workflows/sync.yml`)
-   **触发条件**: 
    -   计划任务: 每 6 小时运行一次 (`0 */6 * * *`)。
    -   手动触发: 允许在该 Action 页面手动点击运行。
-   **动作**:
    -   使用 `aormsby/Fork-Sync-With-Upstream-action`。
    -   上游仓库设置为 `CassiopeiaCode/q2api`。
    -   目标分支为 `main`。
    -   使用 `GITHUB_TOKEN` 进行鉴权和推送合并。

### 2. Docker 构建工作流 (`.github/workflows/docker.yml`)
-   **触发条件**:
    -   `main` 分支的 Push 事件。
-   **动作**:
    -   检出代码。
    -   登录 GitHub Container Registry (GHCR)。
    -   处理镜像名称（转换为小写以符合 Docker 规范）。
    -   构建 Docker 镜像。
    -   推送到 `ghcr.io/<你的用户名>/q2api:latest`。
    -   同时也推送到 `sha-xxxx` 标签以便版本回溯。

## 后续步骤
1.  **提交更改**: 将生成的 `.github` 目录推送到你的 GitHub 仓库。
    ```bash
    git add .github
    git commit -m "Add sync and docker build workflows"
    git push
    ```
2.  **启用 Action**: 
    -   进入 GitHub 仓库的 "Actions" 标签页。
    -   如果 GitHub 提示 "Workflows aren't valid" 或需要允许，请点击确认启用。
3.  **GHCR 设置 (可选)**:
    -   初次推送后，镜像会出现在 GitHub 个人主页的 "Packages" 中。
    -   你可以在 Package settings 中将其设置为 Public（如果是私有仓库但想公开镜像）。

## 验证
-   等待定时任务触发或手动触发 "Sync Upstream" 工作流。
-   观察 Sync 完成后，是否自动触发了 "Docker Build and Push" 工作流。
-   检查 GHCR 中是否有新的镜像产出。
