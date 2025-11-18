# 前端部署文档

## 📋 项目概述

知识可视化系统前端，基于 Vue 3 + Vite + TypeScript 构建，使用 GitHub Actions 自动部署到服务器。

## 🚀 快速开始

### 1. 环境要求

- **Node.js**: ≥20.19.0 || ≥22.12.0
- **Docker**: 用于构建和部署
- **服务器**: 支持Docker的Linux服务器

### 2. 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview
```

## ⚙️ 环境配置

### 环境变量说明

| 变量名              | 描述        | 开发环境默认值                   | 生产环境               |
| ------------------- | ----------- | -------------------------------- | ---------------------- |
| `VITE_API_BASE_URL` | 后端API地址 | `http://localhost:8080`          | 通过GitHub Secrets配置 |
| `VITE_APP_TITLE`    | 应用标题    | `Knowledge Visualization System` | 固定值                 |
| `VITE_APP_ENV`      | 环境标识    | `development`                    | `production`           |

### 配置文件

- `.env.development` - 开发环境配置
- `.env.production` - 生产环境配置

## 🛠️ 部署配置

### GitHub Secrets 设置

在 GitHub 仓库的 Settings → Secrets and variables → Actions 中添加：

```
DOCKER_HUB_PASSWORD=dckr_pat_r3YPqQnVrv4LH8pecShMxBRBGIk
SERVER_HOST=111.228.15.67
SERVER_USER=root
SERVER_SSH_KEY=

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACCceppbgzX6385sOBlyPSr0kzzZ2/OVt4he8+nOp4FCGgAAAJgnLi/rJy4v
6wAAAAtzc2gtZWQyNTUxOQAAACCceppbgzX6385sOBlyPSr0kzzZ2/OVt4he8+nOp4FCGg
AAAEAZ55QsER+LAMy4eAqaMwTYCTfUq8WcUQh7RlUhs1B/j5x6mluDNfrfzmw4GXI9KvST
PNnb85W3iF7z6c6ngUIaAAAAETMwOTA0OTcyMTdAcXEuY29tAQIDBA==
-----END OPENSSH PRIVATE KEY-----

SERVER_PORT=22
VITE_API_BASE_URL=http://111.228.15.67:8080
```

### 服务器准备

1. **创建项目目录**

```bash

```

2. **克隆项目**

```bash
git clone https://github.com/你的用户名/knowledge-visualization-system.git .
```

3. **确保Docker已安装**

```bash
docker --version
```

## 🚢 部署流程

### 自动部署（推荐）

1. **推送代码到master分支**

```bash
git add .
git commit -m "你的提交信息"
git push origin master
```

2. **GitHub Actions自动执行**
   - 安装依赖（跳过husky）
   - 构建项目
   - 构建Docker镜像
   - 推送到Docker Hub
   - 通过SSH部署到服务器

### 手动部署

```bash
# 1. 构建Docker镜像
docker build -t knowledge-viz-frontend:latest \
  --build-arg VITE_API_BASE_URL=http://your-server-domain:8080 \
  -f Dockerfile.frontend .

# 2. 停止旧容器
docker stop knowledge-viz-frontend || true
docker rm knowledge-viz-frontend || true

# 3. 启动新容器
docker run -d --name knowledge-viz-frontend --restart unless-stopped \
  -p 80:80 \
  knowledge-viz-frontend:latest

# 4. 清理旧镜像
docker image prune -f
```

## 🏗️ 构建说明

### Docker多阶段构建

1. **构建阶段**
   - 基于 Node.js 20 Alpine
   - 安装依赖（跳过husky避免Git依赖问题）
   - 构建生产版本

2. **生产阶段**
   - 基于 Nginx Alpine
   - 复制构建产物
   - 配置Nginx

### Nginx配置特性

- **SPA路由支持**: `try_files` 处理前端路由
- **静态资源缓存**: 1年缓存策略
- **Gzip压缩**: 减少传输大小
- **API代理**: `/api/` 请求代理到后端容器

## 🔧 故障排查

### 常见问题

1. **husky安装失败**

   ```
   解决方案: 已通过 HUSKY=0 和 --ignore-scripts 解决
   ```

2. **API请求跨域**

   ```
   检查: nginx.conf中的API代理配置
   确保: 后端容器名称正确（achobeta-forge-app）
   ```

3. **构建失败**

   ```bash
   # 检查Node版本
   node --version

   # 清理依赖重新安装
   rm -rf node_modules package-lock.json
   npm install
   ```

4. **容器无法启动**

   ```bash
   # 查看容器日志
   docker logs knowledge-viz-frontend

   # 检查端口占用
   netstat -tlnp | grep :80
   ```

### 日志查看

```bash
# 查看容器日志
docker logs knowledge-viz-frontend

# 实时监控日志
docker logs -f knowledge-viz-frontend

# 查看nginx错误日志
docker exec knowledge-viz-frontend cat /var/log/nginx/error.log
```

## 🔄 更新流程

1. **代码更新**

   ```bash
   git pull origin master
   ```

2. **重新部署**
   - 推送到master分支触发自动部署
   - 或执行手动部署命令

## 🌐 访问地址

- **开发环境**: http://localhost:5173
- **生产环境**: http://你的服务器IP

## 📁 项目结构

```
├── .env.development          # 开发环境配置
├── .env.production          # 生产环境配置
├── .github/workflows/       # GitHub Actions配置
│   └── frontend-deploy.yml  # 前端部署工作流
├── Dockerfile.frontend      # Docker构建文件
├── nginx.conf              # Nginx配置
├── src/
│   ├── api/                # API接口
│   ├── types/              # 类型定义
│   └── utils/
│       └── request.ts      # Axios封装（已支持环境变量）
└── DEPLOY.md               # 部署文档
```

## 📞 支持

如果遇到问题，请检查：

1. GitHub Actions日志
2. 服务器Docker日志
3. 网络连接状态

---

**注意**: 前端代码打包后完全暴露给用户，不要在前端存储任何敏感信息（API密钥、密码等）。
