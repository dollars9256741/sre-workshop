# 附錄

## 延伸學習資源

- [Docker 官方文件](https://docs.docker.com/)
- [Dockerfile 最佳實踐](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose 規格](https://docs.docker.com/compose/compose-file/)

| 主題 | 說明 |
|------|------|
| **Docker Network 進階** | Bridge、Host、Overlay 等網路模式 |
| **Docker Security** | 映像檔掃描、Rootless 模式、Secret 管理 |
| **Container Registry** | Harbor、ECR、GCR 等私有映像檔倉庫 |
| **Container Orchestration** | Kubernetes、Docker Swarm |
| **CI/CD 整合** | GitHub Actions、GitLab CI 中使用 Docker |
| **多平台建置** | `docker buildx` 建置 ARM/AMD64 映像檔 |
| **日誌管理** | 日誌驅動程式、集中式日誌收集 |
| **監控** | Prometheus + Grafana 監控容器 |

---

## 附錄 A：Docker 指令速查表

### 映像檔 (Image)

```bash
docker pull <image>:<tag>           # 下載映像檔
docker images                        # 列出映像檔
docker rmi <image>                   # 刪除映像檔
docker build -t <name>:<tag> .       # 建置映像檔
docker tag <src> <dest>              # 標記映像檔
docker push <image>:<tag>            # 推送映像檔
docker save -o file.tar <image>      # 匯出映像檔
docker load -i file.tar              # 匯入映像檔
docker history <image>               # 查看建置歷史
docker inspect <image>               # 查看詳細資訊
docker image prune                   # 清理 dangling 映像檔
docker image prune -a                # 清理所有未使用映像檔
```

### 容器 (Container)

```bash
docker run [opts] <image> [cmd]      # 建立並啟動容器
docker create [opts] <image> [cmd]   # 建立容器（不啟動）
docker start <container>             # 啟動容器
docker stop <container>              # 停止容器
docker restart <container>           # 重啟容器
docker kill <container>              # 強制停止
docker rm <container>                # 刪除容器
docker ps                            # 列出執行中容器
docker ps -a                         # 列出所有容器
docker logs [-f] <container>         # 查看日誌
docker exec -it <container> <cmd>    # 在容器內執行命令
docker cp <src> <container>:<dest>   # 複製檔案
docker inspect <container>           # 查看詳細資訊
docker stats                         # 即時資源使用
docker top <container>               # 查看程序
docker port <container>              # 查看連接埠對映
docker container prune               # 清理已停止容器
```

### 資料卷 (Volume)

```bash
docker volume create <name>          # 建立資料卷
docker volume ls                     # 列出資料卷
docker volume inspect <name>         # 查看詳細資訊
docker volume rm <name>              # 刪除資料卷
docker volume prune                  # 清理未使用資料卷
```

### 網路 (Network)

```bash
docker network create <name>         # 建立網路
docker network ls                    # 列出網路
docker network inspect <name>        # 查看詳細資訊
docker network rm <name>             # 刪除網路
docker network connect <net> <ctn>   # 將容器加入網路
docker network disconnect <net> <c>  # 將容器移出網路
docker network prune                 # 清理未使用網路
```

### 系統

```bash
docker system df                     # 磁碟使用量
docker system prune                  # 清理未使用資源
docker system prune -a --volumes     # 深度清理（含映像檔和資料卷）
docker info                          # 系統資訊
docker version                       # 版本資訊
```

---

## 附錄 B：Dockerfile 指令速查表

```dockerfile
# Base image
FROM <image>:<tag> [AS <name>]

# Working directory
WORKDIR /path

# Copy files
COPY [--from=<stage>] <src> <dest>
ADD <src> <dest>

# Execute commands (build stage)
RUN <command>

# Environment variables
ENV KEY=value
ARG KEY=default

# Declare port
EXPOSE <port>

# Container startup command
CMD ["executable", "param1"]
ENTRYPOINT ["executable", "param1"]

# User
USER <user>[:<group>]

# Health check
HEALTHCHECK [OPTIONS] CMD <command>

# Metadata
LABEL key="value"

# Stop signal
STOPSIGNAL signal

# Shell
SHELL ["executable", "parameters"]
```

---

## 附錄 C：Docker Compose 速查表

```bash
docker compose up -d                 # 啟動服務（背景）
docker compose up -d --build         # 啟動並重新建置
docker compose down                  # 停止並移除
docker compose down -v               # 停止、移除、刪除 Volume
docker compose ps                    # 服務狀態
docker compose logs [-f] [service]   # 服務日誌
docker compose exec <svc> <cmd>      # 在服務內執行命令
docker compose run --rm <svc> <cmd>  # 一次性命令
docker compose build [--no-cache]    # 建置映像檔
docker compose pull                  # 拉取映像檔
docker compose restart [service]     # 重啟服務
docker compose config                # 驗證設定
docker compose top                   # 查看服務程序
```

---

## 附錄 D：容器底層技術

容器底層靠的是 Linux 核心的三個機制來做隔離：

**Namespace — 隔離可見範圍**

Namespace 讓每個容器以為自己是系統上唯一的存在，只能看到屬於自己的資源。

| Namespace 類型 | 隔離內容 | 效果 |
|---------------|---------|------|
| **PID** | 程序 ID | 容器內的程序以為自己是 PID 1 |
| **Network** | 網路介面、IP、路由 | 每個容器有獨立的網路堆疊與 IP 位址 |
| **Mount** | 檔案系統掛載點 | 容器只能看到自己的檔案系統 |
| **UTS** | 主機名稱 | 每個容器可以有自己的 hostname |
| **IPC** | 跨程序通訊 | 容器間的 IPC 相互隔離 |
| **User** | 使用者/群組 ID | 容器內的 root 不等同於宿主機的 root |
| **Cgroup** | Cgroup 層級 | 容器只能看到自己的資源限制 |

簡單來說，你在宿主機上跑 `ps aux` 可以看到所有程序，但在容器裡面跑同樣的指令就只能看到自己的程序，這就是 PID Namespace 在做的事。

**Cgroup（Control Group）— 限制資源用量**

Cgroup 管的是每個容器可以用多少系統資源：

| 資源 | 說明 | 範例 |
|------|------|------|
| **CPU** | 限制 CPU 使用量 | 最多使用 1.5 個 CPU 核心 |
| **Memory** | 限制記憶體使用量 | 最多使用 512MB，超過則 OOM Kill |
| **I/O** | 限制磁碟讀寫速度 | 讀寫速度上限 100MB/s |
| **Network** | 限制網路頻寬 | 上行/下行各 100Mbps |

**Union Filesystem — 管理檔案系統**

Union Filesystem（像 OverlayFS）可以把多個目錄「疊加」在一起，看起來像一個統一的檔案系統。這是 Docker 映像檔分層架構的基礎（詳見 [01-docker-concepts.md 1.5 節](01-docker-concepts.md#15-映像檔與容器的關係)）。

```
┌───────────────────────────────────────────────────────┐
│              容器的三大底層技術                          │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │
│  │  Namespace  │  │   Cgroup    │  │ Union FS     │   │
│  │             │  │             │  │              │   │
│  │ 隔離可見性    │  │ 限制資源量   │  │ 管理檔案系統   │   │
│  │ 「看到什麼」  │  │ 「用多少」   │  │ 「存什麼」     │   │
│  └─────────────┘  └─────────────┘  └──────────────┘   │
│         │                │                │           │
│         └────────────────┼────────────────┘           │
│                          ▼                            │
│                 ┌─────────────────┐                   │
│                 │   Linux Kernel  │                   │
│                 └─────────────────┘                   │
│                                                       │
│  這些都是 Linux 核心的原生功能，Docker 的貢獻是將它們       │
│  封裝成簡單易用的工具。                                  │
│                                                       │
└───────────────────────────────────────────────────────┘
```

> **注意**：容器不是真正獨立的虛擬機器，它就是被核心功能隔離出來的程序。你在宿主機上跑 `ps aux` 其實看得到容器裡的程序。它們本質上就是普通的 Linux 程序，只是被 Namespace 跟 Cgroup 圍起來而已。
