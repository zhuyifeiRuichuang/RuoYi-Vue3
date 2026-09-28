# RuoYi-Vue3 CI/CD 与容器化部署配置（cicd/）

本目录与 `.github/workflows/` 下的工作流构成 RuoYi-Vue3 前端的**全手动触发**流水线，配合 RuoYi-Cloud 仓库的 `cicd/` 完成整体发布与部署。

## 工作流（Actions 页手动触发，填写版本号即 Tag 名）

| 工作流 | 作用 |
| --- | --- |
| `release-frontend.yml` | `npm run build:prod`（Node 22 LTS，npmjs.org 官方源）→ 创建 Release 上传 `ruoyi-web-dist.zip` |
| `build-images-frontend.yml` | 下载 dist → 构建多架构前端镜像 → `ghcr.io/zhuyifeiRuichuang/ruoyi-vue3/ruoyi-web` |
| `deploy-test.yml` | 在 amd64 环境下分别验证 docker compose 与标准 k8s（kind）部署 |

**版本号规则**：工作流不会自动添加任何前缀。输入 `dev-3.9.2` 即生成 Release Tag `dev-3.9.2`、镜像 Tag `dev-3.9.2`。请确保 `build-images-frontend` 输入的版本号与 `release-frontend` 完全一致。

## 设计原则

- **零镜像 / 零加速**：`setup-node` 的 `registry-url` 强制指向 `https://registry.npmjs.org`，不使用任何镜像 / 中国加速。
- **最新基础软件**：Node 22 LTS；基础镜像 `nginx:1.27-alpine`（官方、多架构）。
- **多架构**：`build-images-frontend` 通过 buildx + QEMU 构建 `linux/amd64,linux/arm64`，并关闭 provenance 以消除 `unknown/unknown` 冗余 tag。
- **部署测试**：`deploy-test` 提供 docker compose 与 kind 两种 amd64 环境验证。
- **不动上游代码结构**：新增文件仅在 `.github/workflows/` 与 `cicd/`，未改动源码与 `src/` 等。

## 推荐执行顺序（与 RuoYi-Cloud 配合）

1. RuoYi-Cloud：`release-backend`
2. 本仓库：`release-frontend`
3. RuoYi-Cloud：`build-images-backend`
4. 本仓库：`build-images-frontend`
5. RuoYi-Cloud：`deploy-test`（拉取两仓库镜像，验证 compose + k8s 部署）
6. 本仓库：`deploy-test`（验证前端镜像在 compose + k8s 中的部署）

## 权限 / Secret

- `GITHUB_TOKEN`：自动提供，已授予 `contents`/`packages` 写入权限。
- 镜像默认尽力设为公开；若私有，RuoYi-Cloud 仓库的 `deploy-test` 需 `GHCR_PAT` 机密。
