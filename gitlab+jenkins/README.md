## GitLab + Jenkins 一体化 Docker Compose 部署指南

本文档介绍如何使用 Docker Compose 一键部署 GitLab 与 Jenkins，实现自动化 CI/CD 流程。文档分为以下几个部分：

1. 概述
2. 环境准备
3. 项目目录结构
4. 环境下载（镜像拉取）
5. 编写 Docker Compose 文件
6. 部署与启动
7. Jenkins 与 GitLab 初始化
8. 集成配置
9. GitLab Webhook 设置
10. CI/CD 流程验证
11. 可选扩展

------

### 1. 概述

本指南针对私有服务器环境，基于 Docker Compose 将 GitLab（CE 版本）和 Jenkins（LTS 版本）部署在同一机群中，支持通过 GitLab Push 事件自动触发 Jenkins 构建，并可快速扩展到 Docker 构建、镜像 Registry、自动部署等高级场景。

### 2. 环境准备

- **操作系统**：任意支持 Docker 的 Linux 发行版（Ubuntu、CentOS 等）
- **Docker**：建议版本 20.10+
- **Docker Compose**：建议版本 1.29+
- **网络要求**：必要的 HTTP/HTTPS 端口映射
- **硬件配置**：至少 4 GB 可用内存、双核 CPU

### 3. 项目目录结构

```text
gitlab-jenkins/
├── docker-compose.yml       # Docker Compose 配置文件
├── gitlab/
│   ├── config/             # GitLab 配置文件目录（持久化）
│   ├── logs/               # GitLab 日志目录（持久化）
│   └── data/               # GitLab 数据目录（持久化）
└── jenkins_home/           # Jenkins 家目录（Docker 卷）
```

### 4. 环境下载（镜像拉取）

在正式启动前，可手动拉取所需镜像以加快后续启动速度：

```bash
docker pull gitlab/gitlab-ce:17.11.5-ce.0
docker pull jenkins/jenkins:jdk17
 # 可选，用于 GitLab Runner
```

也可跳过此步，直接使用 `docker-compose up` 自动拉取。

### 5. 编写 Docker Compose 文件

在项目根目录创建 `docker-compose.yml`：

```yaml
version: '3.8'

networks:
  gitlab_net:
    driver: bridge

services:
  gitlab:
    image: gitlab/gitlab-ce:17.11.5-ce.0
    container_name: gitlab
    hostname: gitlab
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab:80'
        gitlab_rails['gitlab_shell_ssh_port'] = 22
    ports:
      - '8443:443'
      - '8929:80'
      - '2224:22'
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab
    networks:
      - gitlab_net

  jenkins:
    image: jenkins/jenkins:jdk17
    container_name: jenkins
    restart: always
    ports:
      - '8081:8080'
      - '50000:50000'
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitlab_net
```

> **说明**：
>
> - GitLab 映射宿主机端口 8929（HTTP）和 2224（SSH）。
> - Jenkins 映射 8081（Web 控制台）和 50000（Agent 通信）。

### 6. 部署与启动

切换到项目根目录，执行：

```bash
docker network create gitlab_net

docker-compose up -d
```

确认服务状态：

```bash
docker-compose ps
```

若一切正常，GitLab 与 Jenkins 容器应处于 `Up` 状态。

### 7. Jenkins 与 GitLab 初始化

#### 7.1 Jenkins

1. 浏览器访问 `http://<服务器IP>:8081`。

2. 获取初始管理员密码：

   ```bash
   docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

3. 按提示安装推荐插件并创建管理员用户。

#### 7.2 GitLab

1. 浏览器访问 `http://<服务器IP>:8929`。
2. 初始登录账号为 `root`，设置密码并登录。
3. 创建用户与项目仓库。


### 8. 集成配置

1. **安装 Jenkins 插件**：
- Git plugin
- GitLab Plugin
- Pipeline
- GitLab Hook Plugin（可选）

2. **创建 GitLab 访问令牌**：
- GitLab → 用户设置 → Access Tokens
- 名称 `jenkins-token`，勾选 `api`、`read_repository`、`write_repository`

3. **在 Jenkins 中添加凭据**：
- Jenkins → Manage Jenkins → Credentials
- 秘密文本（Secret Text），ID 填 `gitlab_token`，内容填写刚创建的 Token

4. **创建 Pipeline 任务**：
- 新建任务，类型选 `Pipeline`
- 配置：
  - SCM 选 Git，仓库 URL 填 `http://gitlab.local:8929/<namespace>/<project>.git`
  - 凭据选 `gitlab_token`
  - 指定分支，例如 `main`
  - Pipeline 脚本可从 SCM 加载 `Jenkinsfile`


### 9. GitLab Webhook 设置

1. 进入 GitLab 项目 → Settings → Webhooks
2. URL 填：http://<服务器IP>:8081/project/



3. 触发事件勾选 **Push events**（也可选 Merge request events）
4. 可选：填写 Secret Token，同步 Jenkins Hook 插件配置。


### 10. CI/CD 流程验证

1. 在本地或 GitLab Web IDE 编辑项目，提交并 Push 到 GitLab。
2. 观察 Jenkins 控制台，自动触发构建。
3. 构建通过后，将执行 `Jenkinsfile` 中定义的构建与部署步骤。


### 11. 可选扩展

| 功能            | 实现方式                                  |
| --------------- | ----------------------------------------- |
| Docker 镜像构建 | 在 Jenkinsfile 中使用 `docker build`      |
| 构建产物托管    | Artifactory、Nexus 或自建 Registry        |
| 自动化部署      | 使用 `ssh`、`kubectl`、`ansible` 等       |
| 多环境参数化    | Jenkins 参数化构建 + 环境变量管理         |
| GitLab Runner   | 增加 `gitlab-runner` 服务，实现更灵活执行 |

