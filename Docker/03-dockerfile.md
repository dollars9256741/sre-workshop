# 03 — Dockerfile 基礎

## 3.1 什麼是 Dockerfile？

Ocean 用 Compose 把服務跑得有聲有色，但有一天 Andrew 推了新的 Go 程式碼，Ocean 發現 Docker Hub 上根本沒有現成的映像檔可以用。Andrew：「別人的映像檔當然沒有我們的程式碼。要把自己的程式打包成映像檔，就要寫 Dockerfile。」

到目前為止，我們用的映像檔（`nginx`、`postgres`）都是別人做好放在 Docker Hub 上的。但如果你要跑的是自己寫的程式，就需要自己做一個映像檔。

做映像檔的方式就是寫一個 **Dockerfile**。它是一個純文字檔，放在專案的根目錄，裡面一行一行寫著建置映像檔的步驟，像是：用哪個基礎環境、複製哪些檔案進去、執行什麼安裝指令、程式怎麼啟動。

你可以把 Dockerfile 想成是一份「環境建置的 SOP」。手動裝環境的時候，你可能會：

1. 找一台乾淨的 Linux 機器
2. 安裝 Go 語言
3. 把程式碼複製過去
4. 編譯程式
5. 設定啟動指令

Dockerfile 做的就是把這些步驟寫下來，讓 Docker 自動執行。寫好之後跑一個 `docker build` 指令，Docker 就會照著步驟一步一步做，最後產出一個映像檔：

```
Dockerfile ─── docker build ──→ Docker Image ─── docker run ──→ Container
（建置腳本）      （建置過程）      （成品）          （執行）       （跑起來的容器）
```

這個映像檔可以推到 Docker Hub 或私有 Registry，讓其他人拉下來直接用，不需要重新建置。

---

## 3.2 Dockerfile 基礎指令

假設你有一個最簡單的 Go 專案，結構長這樣：

```
my-app/
├── main.go         # Go 程式碼
├── go.mod          # Go 模組定義
└── Dockerfile      # 建置映像檔的腳本
```

`main.go` 是一個簡單的 HTTP Server，`go.mod` 是 Go 的套件管理檔，而 `Dockerfile` 就是我們要寫的東西。來看看這個 Dockerfile 長什麼樣：

```dockerfile
FROM golang:1.24-alpine
WORKDIR /app
COPY . .
RUN go build -o server .
CMD ["./server"]
```

只有五行，但做了完整的事情。一行一行來看：

**`FROM golang:1.24-alpine`** — 起點。每個 Dockerfile 都從 `FROM` 開始，指定你要在什麼基礎環境上建置。這裡用的是已經裝好 Go 1.24 的 Alpine Linux。

**`WORKDIR /app`** — 設定映像檔內部的工作目錄。後面的 `COPY`、`RUN` 都會在 `/app` 這個目錄下執行。

**`COPY . .`** — 把你本機目前目錄下的所有檔案複製到映像檔的 `/app` 裡面。第一個 `.` 是本機路徑，第二個 `.` 是容器內的路徑（也就是 `WORKDIR` 設定的 `/app`）。

**`RUN go build -o server .`** — 在建置過程中執行命令。這裡是編譯 Go 程式，產出一個叫 `server` 的執行檔。`RUN` 的結果會被寫進映像檔的 Layer 裡。

**`CMD ["./server"]`** — 容器啟動時的預設命令。當你 `docker run` 這個映像檔的時候，它就會執行 `./server`。

建置並執行：

```bash
docker build -t my-app .     # 用目前目錄的 Dockerfile 建置映像檔，取名叫 my-app
docker run -p 8080:8080 my-app   # 跑起來
```

### 指令速查表

上面的範例只用了五個指令，但 Dockerfile 還有其他常用指令。完整列表如下：

| 指令 | 說明 | 範例 |
|------|------|------|
| `FROM` | 指定基礎映像檔（必須是第一行） | `FROM golang:1.24-alpine` |
| `WORKDIR` | 設定工作目錄 | `WORKDIR /app` |
| `COPY` | 複製檔案至映像檔 | `COPY . .` |
| `RUN` | 建置時執行命令 | `RUN go build -o server .` |
| `CMD` | 容器啟動時的預設命令 | `CMD ["./server"]` |
| `ENTRYPOINT` | 容器啟動時的固定入口 | `ENTRYPOINT ["./server"]` |
| `EXPOSE` | 宣告容器監聽的連接埠（僅文件用途） | `EXPOSE 8080` |
| `ENV` | 設定環境變數 | `ENV GIN_MODE=release` |
| `USER` | 指定執行命令的使用者 | `USER nonroot` |
| `HEALTHCHECK` | 定義健康檢查 | `HEALTHCHECK CMD curl -f http://localhost/` |

---

## 3.3 為 Go 應用撰寫 Dockerfile

先來看一個簡單的 Go HTTP 伺服器範例：

**main.go**

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		hostname, _ := os.Hostname()
		fmt.Fprintf(w, "Hello from Go! Hostname: %s\n", hostname)
	})

	http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		fmt.Fprint(w, "OK")
	})

	log.Printf("Server starting on port %s", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

### Multi-stage Build

3.2 的五行 Dockerfile 已經能跑了，但有個問題：最終映像檔裡面包含了整個 Go 編譯工具鏈（~300MB+），跑起來的時候根本只需要那個編譯好的二進位檔。Ocean 照著 3.2 的方式 build 出來一看，映像檔 300 多 MB。Andrew 看到差點把咖啡噴出來：「你要拿這個上正式環境？裡面一整套 Go 編譯工具鏈都打包進去了啊。」Ocean：「那要怎麼辦？」Andrew：「用 Multi-stage Build，build 完只留下執行檔就好。」

Multi-stage Build（多階段建置）的概念很簡單：**建置階段**用完整的工具鏈編譯程式，**最終映像檔**只放跑起來需要的東西。

```dockerfile
# ==========================================
# Stage 1: Build（建置階段）
# ==========================================
FROM golang:1.24-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-s -w" \
    -o server .

# ==========================================
# Stage 2: Runtime（執行階段）
# ==========================================
FROM alpine:3.21

RUN apk add --no-cache ca-certificates tzdata \
    && addgroup -S appgroup \
    && adduser -S appuser -G appgroup

WORKDIR /app

# 從 builder 階段複製編譯好的二進位檔
COPY --from=builder /app/server .

USER appuser
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

ENTRYPOINT ["./server"]
```

**映像檔大小比較：**

| 方式 | 映像檔大小 |
|------|-----------|
| 單階段 `golang:1.24-alpine` | ~300MB |
| 多階段 `alpine:3.21` | ~15MB |
| 多階段 `scratch`（最精簡） | ~8MB |

### 建置與執行

```bash
# 建置映像檔
docker build -t my-go-app:v1 .

# 查看映像檔大小
docker images my-go-app

# 執行容器
docker run -d --name my-app -p 8080:8080 my-go-app:v1

# 測試
curl http://localhost:8080
curl http://localhost:8080/health
```

---

→ 綜合練習：[exercises/02-debug-compose.md](exercises/02-debug-compose.md)
