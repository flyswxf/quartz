# Docker 核心完全指南 🐳

> "Build once, run anywhere." —— 一次构建，到处运行。

## 1. 什么是 Docker？形象理解 💡

想象你在搬家：
*   **传统方式**：你需要把家具、衣服、锅碗瓢盆一件件搬到新家，到了新家还得重新组装柜子、重新接电视线、重新摆放物品。如果在搬运过程中丢了一个螺丝，柜子可能就装不上了。（**环境配置地狱**）
*   **Docker 方式**：你有一个神奇的**集装箱**。你在旧家把所有东西都摆好，封进集装箱里。到了新家，把集装箱往地上一放，打开门，里面的东西和你旧家一模一样，直接可以住人。（**容器化**）

### 核心概念映射

| 概念 | 英文 | 形象比喻 | 解释 |
| :--- | :--- | :--- | :--- |
| **镜像** | **Image** | **应用的光盘/模具** | 只读的模板。比如 Windows 安装盘，或者做蛋糕的模具。它包含了运行程序所需的一切（代码、库、环境变量、配置文件）。 |
| **容器** | **Container** | **运行中的实体/蛋糕** | 镜像的运行实例。用模具（Image）烤出来的蛋糕（Container）。你可以用同一个模具烤出无数个一模一样的蛋糕。容器是可以被启动、停止、删除的。 |
| **仓库** | **Registry** | **应用超市/App Store** | 存放镜像的地方。比如 Docker Hub。你可以从这里下载（pull）别人做好的镜像，也可以上传（push）自己的。 |
| **数据卷** | **Volume** | **外挂硬盘/U盘** | 容器里的数据默认是临时的，容器一删数据就没了。Volume 就像插在容器上的 U盘，把数据保存在宿主机上，防止数据丢失。 |
| **Dockerfile** | **Dockerfile** | **自动构建说明书** | 一个文本文件，告诉 Docker 怎么一步步制作出那个“镜像”。比如：1. 拿一个空盘子; 2. 放上面粉; 3. 加水... |

---

## 2. 核心架构图解 🖼️

```mermaid
graph LR
    subgraph Client
        DockerCLI[Docker CLI<br>(docker build/run)]
    end

    subgraph Docker_Host
        Daemon[Docker Daemon<br>(dockerd)]
        subgraph Containers
            C1[Container 1]
            C2[Container 2]
        end
        subgraph Images
            I1[Image: Redis]
            I2[Image: Nginx]
        end
    end

    subgraph Registry
        Hub[Docker Hub]
    end

    DockerCLI -->|REST API| Daemon
    Daemon -->|pull| Hub
    Daemon -->|run| C1
    I1 -.-> C1
```

---

## 3. 常用命令速查 (Cheatsheet) ⚡

### 基础生命周期
*   `docker pull <image>`: 下载镜像 (去超市买模具)
*   `docker run <image>`: 启动容器 (用模具做蛋糕)
    *   `-d`: 后台运行 (BackgrounD)
    *   `-p 8080:80`: 端口映射 (宿主机端口:容器端口)
    *   `--name my-app`: 给容器起个名字
    *   `-v /host/path:/container/path`: 挂载数据卷
*   `docker ps`: 查看正在运行的容器
*   `docker stop <container_id>`: 停止容器
*   `docker rm <container_id>`: 删除容器 (删之前通常要先 stop)
*   `docker rmi <image_id>`: 删除镜像

### 调试与交互
*   `docker exec -it <container_id> /bin/bash`: **进入**正在运行的容器内部 (就像通过传送门进入集装箱内部修东西)
    *   `-i`: 交互式 (Interactive)
    *   `-t`: 伪终端 (Tty)
*   `docker logs <container_id>`: 查看容器日志 (排错神器)
    *   `-f`: 实时跟踪日志 (Follow)
*   `docker inspect <container_id>`: 查看容器详细信息 (IP地址、挂载点等)

---

## 4. Dockerfile 最佳实践 (Best Practices) 🛠️

这是 Docker 的核心。写好 Dockerfile 决定了你的镜像大小、构建速度和安全性。

### 示例：Python Flask 应用

```dockerfile
# 1. 基础镜像：选择轻量级的 (Alpine 或 Slim)
# ❌ FROM ubuntu:latest (太大，包含很多不需要的东西)
# ✅ FROM python:3.9-slim
FROM python:3.9-slim

# 2. 工作目录
WORKDIR /app

# 3. 利用缓存机制 (Layer Caching)
# 先拷贝 requirements.txt，再安装依赖。
# 只要 requirements.txt 没变，Docker 就会直接使用缓存层，跳过 pip install，大大加快构建速度。
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. 拷贝源代码
COPY . .

# 5. 非 Root 用户 (Security)
# 默认容器以 root 运行，有安全风险。创建一个普通用户来运行应用。
RUN useradd -m myuser
USER myuser

# 6. 暴露端口
EXPOSE 5000

# 7. 启动命令
CMD ["python", "app.py"]
```

### 关键原则解释
1.  **最小化基础镜像**: 优先使用 `alpine` (极小 Linux) 或 `slim` 版本。镜像越小，下载越快，攻击面越小。
2.  **分层缓存 (Layer Caching)**: Docker 是分层构建的。把**不常变**的指令（如安装依赖）放在前面，**常变**的指令（如 COPY 代码）放在后面。
3.  **多阶段构建 (Multi-stage Builds)**: 编译型语言（Go, Java, C++）的神器。
    *   *场景*: 编译代码需要很多工具（gcc, maven），但运行代码只需要一个二进制文件。
    *   *做法*: 第一阶段用全量镜像编译，第二阶段只拷贝编译好的文件到精简镜像。

    ```dockerfile
    # 阶段 1: 构建 (Builder)
    FROM golang:1.19 AS builder
    WORKDIR /app
    COPY . .
    RUN go build -o myapp main.go

    # 阶段 2: 运行 (Runner)
    FROM alpine:latest
    WORKDIR /root/
    # 只从 builder 阶段拿编译好的文件，丢弃源代码和编译器
    COPY --from=builder /app/myapp .
    CMD ["./myapp"]
    ```

---

## 5. Docker Compose: 编排多个容器 🎼

现实中，一个应用通常包含：Web 服务 + 数据库 + 缓存。用 `docker run` 一个个启动太麻烦。
`docker-compose.yml` 就像一个乐谱，指挥所有容器一起工作。

### 示例：Web (Python) + Redis

```yaml
version: "3.8"  # 版本号

services:
  web:
    build: .             # 使用当前目录的 Dockerfile 构建
    ports:
      - "5000:5000"
    volumes:
      - .:/code          # 开发环境神器：代码热更新 (挂载当前目录)
    environment:
      - FLASK_ENV=development
    depends_on:
      - redis            # 等 redis 启动了再启动 web

  redis:
    image: "redis:alpine"
```

*   **一键启动**: `docker-compose up -d`
*   **一键停止**: `docker-compose down`

---

## 6. 不同情景下的最佳实践 🎯

### 👨‍💻 开发环境 (Development)
*   **挂载代码 (Bind Mounts)**: 使用 `-v` 或 Compose 中的 `volumes` 将本地代码映射进容器。这样你修改本地代码，容器内立即生效，无需重新构建镜像。
*   **调试**: 开启 Debug 模式，使用 `docker exec` 进入容器调试。
*   **工具链**: 可以把所有的开发工具（编译器、Linter）都放进 Docker，保证团队成员开发环境 100% 一致（不再有 "It works on my machine"）。

### 🚀 生产环境 (Production)
*   **镜像不可变 (Immutable)**: 代码应该打进镜像里（COPY），**不要**像开发环境那样挂载代码目录。确保测试过的镜像和上线的是同一个。
*   **配置分离**: 使用环境变量 (`-e` 或 `.env` 文件) 传递敏感信息（密码、API Key），不要写死在 Dockerfile 里。
*   **资源限制**: 使用 `--memory` 和 `--cpus` 限制容器资源，防止一个容器吃光服务器内存导致死机。
*   **健康检查 (Healthcheck)**: 在 Dockerfile 或 Compose 中定义 `HEALTHCHECK`，让 Docker 知道你的服务是不是还活着。
*   **只读文件系统**: 尽可能让容器以 Read-only 模式运行，提高安全性。

### 🛡️ 安全 (Security)
*   **不要用 Root**: 永远不要以 root 身份运行生产环境的容器进程。
*   **定期更新**: 经常更新基础镜像，修补系统漏洞。
*   **最小权限**: 只映射需要的端口，只挂载需要的目录。
