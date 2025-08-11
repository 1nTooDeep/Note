# Dify本地部署

## 1. 准备工作
首先，你需要确保自己拥有运行Dify的环境.这里使用Docker Compose部署Dify.
> 在安装 Dify 之前，请确保您的设备符合以下最低系统要求：\
CPU >= 2 核 \
RAM >= 4 GiB
1. 运行环境：Docker、Docker Compose
## 2. 克隆仓库
```bash
git clone https://github.com/langgenius/dify.git
```
## 3. 启动Dify
确保当前工作目录为dify
```bash
cp middleware.env.example middleware.env
docker compose -f docker-compose.middleware.yaml up -d