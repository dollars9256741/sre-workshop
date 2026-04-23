# 練習 1：執行你的第一個容器

> 搭配章節：[01-docker-concepts.md](../01-docker-concepts.md)

Nginx 是一個常見的網頁伺服器，很多網站背後都用它來處理請求，這次的目標就是啟動一個 Nginx 容器，掛載自訂頁面，並透過瀏覽器存取。

## Step 1：建立工作目錄和網頁

```bash
mkdir -p ~/docker-lab/lab1
cd ~/docker-lab/lab1

cat << 'EOF' > index.html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Lab 1</title>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>This page is served from a Docker container.</p>
</body>
</html>
EOF
```

## Step 2：啟動 Nginx 容器

```bash
docker run -d \
  --name lab1-nginx \
  -p 8080:80 \
  -v $(pwd):/usr/share/nginx/html \
  nginx:1.27-alpine
```

## Step 3：確認容器在跑，然後測試

```bash
docker ps
curl http://localhost:8080
```

打開瀏覽器連 `http://localhost:8080`，應該會看到你剛才寫的 HTML 頁面。

## Step 4：試試除錯指令

```bash
# 看日誌
docker logs lab1-nginx

# 進去容器裡面逛逛
docker exec -it lab1-nginx sh
# 試試看在裡面：
#   ls /usr/share/nginx/html/
#   cat /etc/nginx/nginx.conf
#   exit # 離開容器，回到自己的終端機。
```

## Step 5：清理

```bash
docker stop lab1-nginx
docker rm lab1-nginx
```

## 延伸練習

1. 修改 `index.html` 的內容，不重啟容器，直接重新整理瀏覽器查看變化（思考：為什麼修改會立即生效？）
