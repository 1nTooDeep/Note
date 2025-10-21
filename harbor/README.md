# Harbor

## Harbor 备份

### 1. 服务器之间备份

当你需要在服务器(相同版本)之间迁移Harbor镜像数据时，最好用自带的复制功能进行镜像复制
   1. 配置源机器人账户，记住AK和TOKEN
   2. 由于可能配置harbor的仓库域名如`dockerhub.kubekey.local`，而这个域名在内
      网环境下可能无法解析，所以需要将这个域名设置到容器的配置文件`/etc/hosts`中
      ```bash
      docker exec --user=root harbor-core bash -c "echo '192.168.0.10 dockerhub.kubekey.local' >> /etc/hosts"
      docker exec --user=root harbor-jobservice bash -c "echo '192.168.0.10 dockerhub.kubekey.local' >> /etc/hosts"
      ```
      
### 2. 单机数据备份
  进行单机备份时，必须要停止运行中的harbor服务：
  ```shell
  docker-compose down
  ```
  备份脚本：
  ```shell
#!/bin/bash
#
# Harbor backup script
# 用于安全备份 Harbor（docker-compose 部署版本）
# 功能：
#   1. 检测 harbor 是否在运行；
#   2. 若运行则提示用户是否先停止；
#   3. 默认备份至 /var/backup/harbor/YYYY-MM-DD_HH-mm；
#   4. 使用 tar.gz 最大压缩；
#   5. 备份 harbor.yml、数据库、镜像、chart、证书等核心数据。

set -e

# === 参数与默认值 ===
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")
DEFAULT_BACKUP_DIR="/var/backup/harbor/${TIMESTAMP}"
BACKUP_DIR="${1:-$DEFAULT_BACKUP_DIR}"
HARBOR_DIR="$(cd "$(dirname "$0")" && pwd)"
HARBOR_DOCKER_COMPOSE="$HARBOR_DIR/docker-compose.yml"
PG_CONTAINER="harbor-db"

# === 功能函数 ===

check_docker_running() {
    docker info >/dev/null 2>&1 || {
        echo "❌ Docker 未运行，请先启动 Docker。"
        exit 1
    }
}

check_harbor_running() {
    local running_containers
    running_containers=$(docker compose ps -q 2>/dev/null || true)
    if [ -n "$running_containers" ]; then
        echo "⚠️ 检测到 Harbor 正在运行。"
        echo "建议停止 Harbor 以确保备份一致性。"
        read -p "是否停止 Harbor 以继续备份？(y/n): " confirm
        if [[ "$confirm" =~ ^[Yy]$ ]]; then
            echo "➡️ 停止 Harbor 服务..."
            docker compose down
        else
            echo "❌ Harbor 未停止，取消备份以避免数据不一致。"
            exit 1
        fi
    fi
}

backup_data() {
    echo "📦 正在创建备份目录：$BACKUP_DIR"
    mkdir -p "$BACKUP_DIR"

    echo "🗄️ 备份 harbor 配置文件与元数据..."
    cp -a "$HARBOR_DIR/harbor.yml" "$BACKUP_DIR/" 2>/dev/null || true
    cp -a "$HARBOR_DIR/common" "$BACKUP_DIR/" 2>/dev/null || true
    cp -a "$HARBOR_DOCKER_COMPOSE" "$BACKUP_DIR/" 2>/dev/null || true

    echo "🗃️ 备份 harbor 数据目录..."
    tar --use-compress-program="pigz -9" -cpf "$BACKUP_DIR/harbor_data.tar.gz" \
        /mnt/registry/registry \
        /mnt/registry/database \
        /mnt/registry/chart_storage \
        /mnt/registry/secret 2>/dev/null || {
        echo "⚠️ 文件压缩失败，尝试使用 gzip..."
        tar czf "$BACKUP_DIR/harbor_data.tar.gz" \
            /mnt/registry/registry \
            /mnt/registry/database \
            /mnt/registry/chart_storage \
            /mnt/registry/secret
    }

    echo "🧩 导出 PostgreSQL 数据库快照..."
    if docker ps -a --format '{{.Names}}' | grep -q "^${PG_CONTAINER}$"; then
        docker exec -t "$PG_CONTAINER" pg_dumpall -U postgres > "$BACKUP_DIR/harbor_db_${TIMESTAMP}.sql"
    else
        echo "⚠️ 未找到数据库容器 ${PG_CONTAINER}，跳过 SQL 备份。"
    fi

    echo "✅ Harbor 备份完成！"
    echo "📁 备份路径：$BACKUP_DIR"
    echo "💾 压缩包大小：$(du -sh "$BACKUP_DIR" | awk '{print $1}')"
}

# === 主执行流程 ===
echo "========== Harbor Backup Script =========="
check_docker_running
check_harbor_running
backup_data
echo "=========================================="

  ``` 