# 自动构建并发布 Docker Hub

在 GitHub 仓库的 **Settings → Secrets and variables → Actions** 中设置：

| 类型 | 名称 | 内容 |
| --- | --- | --- |
| Secret | `DOCKER_HUB_USERNAME` | Docker Hub 用户名 |
| Secret | `DOCKER_HUB_TOKEN` | 对目标镜像仓库有写权限的 Docker Hub access token |
| Variable（可选） | `DOCKER_HUB_IMAGE` | 完整镜像名，例如 `yourname/3x-ui-no-ad`，不含标签 |

默认镜像名为 `<DOCKER_HUB_USERNAME>/3x-ui-no-ad`（仓库重命名后使用新的仓库名）。使用组织命名空间时，通过 `DOCKER_HUB_IMAGE` 指定镜像名，并确保 token 有写权限。不要把 token 提交到代码中。

如 fork 的 Actions 尚未启用，先在 **Actions** 页面启用工作流。

`.github/workflows/docker.yml` 在以下情况构建并推送：

- 任意分支发生推送，包括 GitHub 网页 **Sync fork → Update branch** 同步上游产生的新提交。
- 推送 `v*.*.*` 版本标签。
- 在 Actions 页面手动运行 **Build and publish Docker Hub image**。

默认分支生成 `latest`，每个分支生成分支标签，每次运行生成 `sha-<短提交号>`；版本标签还生成原始标签和不带 `v` 的版本号。只有默认分支更新 `latest`。

同步操作没有新提交时不会构建。若另一个 Action 使用 `GITHUB_TOKEN` 推送同步提交，GitHub 不会因此触发新的 push 工作流；该同步流程应使用具有相应权限的 PAT/GitHub App token，或在同步完成后显式触发此工作流的 `workflow_dispatch`。本工作流不负责定时拉取或合并上游代码。

镜像沿用现有的五种 Linux 架构构建，并启用 GitHub Actions 构建缓存。首次多架构编译较慢。同一分支再次推送时会取消尚未完成的旧构建。
