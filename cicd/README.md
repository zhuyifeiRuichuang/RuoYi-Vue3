# RuoYi-Vue3 CI/CD 与容器化部署配置（cicd/）

本目录与 `.github/workflows/` 下的工作流构成 RuoYi-Vue3 前端的**全手动触发**流水线，配合 RuoYi-Cloud 仓库的 `cicd/` 完成整体发布与部署。

## 工作流（Actions 页手动触发，填写版本号，不含 v 前缀）

| 工作流 | 作用 |
| --- | --- |
| `release-frontend.yml` | `npm run build:prod`（Node 22 LTS，npmjs.org 官方源）→ 创建 Release 上传 `ruoyi-web-dist.zip` |
| `build-images-frontend.yml` | 下载 dist → 构建多架构前端镜像 → `ghcr.io/zhuyifeiRuichuang/ruoyi-vue3/ruoyi-web` |

## 设计原则

- **零镜像 / 零加速**：`setup-node` 的 `registry-url` 强制指向 `https://registry.npmjs.org`，不使用任何镜像 / 中国加速。
- **最新基础软件**：Node 22 LTS；基础镜像 `nginx:1.27-alpine`（官方、多架构）。
- **多架构**：`build-images-frontend` 通过 buildx + QEMU 构建 `linux/amd64,linux/arm64`。
- **不动上游代码结构**：新增文件仅在 `.github/workflows/` 与 `cicd/`，未改动源码与 `src/` 等。

## 推荐执行顺序（与 RuoYi-Cloud 配合）

1. RuoYi-Cloud：`release-backend`
2. 本仓库：`release-frontend`
3. RuoYi-Cloud：`build-images-backend`
4. 本仓库：`build-images-frontend`
5. RuoYi-Cloud：`deploy-test`（拉取两仓库镜像，验证 compose + k8s 部署）

## 权限 / Secret

- `GITHUB_TOKEN`：自动提供，已授予 `contents`/`packages` 写入权限。
- 镜像默认尽力设为公开；若私有，RuoYi-Cloud 仓库的 `deploy-test` 需 `GHCR_PAT` 机密。
