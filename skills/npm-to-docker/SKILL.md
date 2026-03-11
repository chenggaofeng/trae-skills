---
name: "npm-to-docker"
description: "将npm全局安装的应用容器化为Docker镜像。当用户需要将已通过npm全局安装的Node.js应用迁移到Docker容器时调用此skill。"
---

# npm全局应用Docker化

将npm全局安装的Node.js应用制作成Docker镜像并迁移部署。

## 适用场景

- 已通过 `npm install -g <package>` 全局安装的应用
- 需要容器化以便于部署、迁移和管理
- 需要保持现有配置和数据

## 实施步骤

### 1. 环境调研

```bash
# 查找安装位置和版本
which <app-name>
npm list -g <package-name>

# 查找配置和数据目录
ls -la ~/.<app-name>/

# 查看端口占用
ss -tlnp | grep <app-name>

# 查看进程
ps aux | grep <app-name>
```

### 2. 创建Dockerfile

```dockerfile
FROM node:22-bookworm-slim

LABEL maintainer="<maintainer>"
LABEL description="<App Description>"

# 安装必要依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    curl \
    ca-certificates \
    git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 全局安装应用
RUN npm install -g <package-name>@<version>

# 创建数据目录
RUN mkdir -p /root/.<app-name> && chmod 700 /root/.<app-name>

ENV NODE_ENV=production

# 暴露端口
EXPOSE <port>

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://127.0.0.1:<port>/health || exit 1

CMD ["<entry-command>"]
```

### 3. 创建docker-compose.yml

```yaml
services:
  <app-name>:
    build:
      context: .
      dockerfile: Dockerfile
    image: <app-name>:<version>
    container_name: <app-name>
    restart: unless-stopped
    volumes:
      - /opt/<app-name>-data:/root/.<app-name>
    ports:
      - "127.0.0.1:<port>:<port>"
    environment:
      - NODE_ENV=production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:<port>/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 4. 数据迁移

```bash
# 创建数据目录
mkdir -p /opt/<app-name>-data

# 复制配置和数据
cp -a ~/.<app-name>/* /opt/<app-name>-data/
```

### 5. 构建和部署

```bash
# 构建镜像
docker build -t <app-name>:<version> .

# 启动容器
docker compose up -d

# 验证状态
docker ps | grep <app-name>
docker logs <app-name>
```

### 6. 清理原服务

```bash
# 备份原配置
tar -czvf ~/backup-$(date +%Y%m%d).tar.gz ~/.<app-name>/

# 停止原服务进程
pkill -f <app-name>

# 卸载npm全局安装
npm uninstall -g <package-name>

# 删除原配置目录
rm -rf ~/.<app-name>/
```

## 注意事项

1. **镜像选择**: 优先使用debian系镜像(node:xx-bookworm-slim)，alpine可能有glibc兼容问题
2. **端口绑定**: 使用 `127.0.0.1:port` 限制外部访问
3. **数据持久化**: 必须挂载volume，否则容器重启数据丢失
4. **启动命令**: 确认正确的入口命令，可能不是包名
5. **依赖安装**: 某些包需要git，记得在Dockerfile中安装

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| npm install失败 | `apt-get install git` |
| native模块编译失败 | 使用debian系镜像 |
| 容器启动失败端口占用 | 先停止原服务进程 |
| 健康检查失败 | 确认正确的health endpoint |
