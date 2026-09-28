# 前端运行时镜像：nginx 托管 dist 静态资源，并反向代理 /prod-api 到网关
# 基础镜像使用国外原版：nginx:1.27-alpine（官方、多架构 amd64/arm64）
FROM nginx:1.27-alpine

COPY dist /usr/share/nginx/html
COPY cicd/docker/nginx/default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
