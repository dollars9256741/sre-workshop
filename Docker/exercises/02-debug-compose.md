# 練習 2：找出壞掉的 Docker Compose

> 搭配章節：[03-dockerfile.md](../03-dockerfile.md)

Andrew 寫了一個 `docker-compose.yml` 想把服務跑起來，但不管怎麼 `docker compose up` 都有東西壞掉。他把檔案丟給 Ocean：「幫我看一下哪裡寫錯了」。

這個 Compose 檔裡有三個服務：`web`（Nginx）、`api`（Alpine）、`db`（PostgreSQL），但藏了**三個設定錯誤**。你的任務是把它們全部找出來並修好。

## Step 1：取得練習檔案

```bash
cd exercises/debug
```

## Step 2：啟動服務，觀察發生什麼事

```bash
docker compose up -d
docker compose ps
```

你會發現有些服務不在 `running` 狀態。

## Step 3：開始排查

以下是你會用到的排查指令：

```bash
# 查看某個服務的日誌
docker compose logs <service-name>

# 查看容器退出碼
docker inspect --format='{{.State.ExitCode}}' <container-name>

# 查看網路內有哪些容器
docker network inspect <network-name>

# 進入容器內測試連線
docker exec -it <container-name> sh
```

## Step 4：修好 `docker-compose.yml`，重新啟動

```bash
docker compose down
# 修改 docker-compose.yml ...
docker compose up -d
docker compose ps
```

反覆排查與修正，直到所有服務都正常運作。

## Step 5：驗證結果

```bash
# web 應該回傳 200
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080

# api 的日誌應該顯示成功取得 web 的回應
docker compose logs api

# db 應該在正常運作
docker compose exec db pg_isready
```

## Step 6：清理

```bash
docker compose down
```

> 提示：三個錯誤分別跟「連接埠」、「網路」、「環境變數」有關。如果卡住了，可以參考 `answer/docker-compose.yml`。
