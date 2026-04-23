# 最後練習：CI/CD + 監控整條串起來

## 課程簡介

25 分鐘的綜合練習，把 Docker、CI/CD、Prometheus 三項技能串在一起。你會透過 GitHub Actions 把服務部署到社辦機器，把 metric 吐給 Prometheus，並在服務壞掉時收到 Discord 告警。

**這個 repo 已經包含：**

- 服務的 source code 和 Dockerfile
- 大部分的 GitHub Actions workflow（CI 完整，CD 還缺一塊）
- Prometheus scrape 設定
- Alert rule

**社辦機器上已經準備好：**

- GitHub self-hosted runner
- Prometheus 和 Alertmanager（Alertmanager 已經設定好對應的 Discord 頻道）

## 適用對象

- 已經上完當天的 Docker、CI/CD、Prometheus 三個工作坊
- 準備好把三個部分合在一起實際跑一次

## 練習流程

### Part 0 — 介紹練習（5 分鐘）

目標架構：repo → GitHub Actions → self-hosted runner → 在社辦機器上 `docker run` → Prometheus scrape → Alertmanager → Discord。

### Part 1 — 在本地跑起來（10 分鐘）

1. `docker compose up` 在本地 build 並啟動服務。
2. 確認 compose 裡面的 Prometheus 有成功 scrape 到服務。
3. 打開 Prometheus UI 找到服務的 metric。
4. 打 `/crash` endpoint，觀察 metric 次數變化。

### Part 2 — 補完 CD，觸發真正的告警（10 分鐘）

1. 參考 CI/CD 工作坊 ch04 的做法，把 CD job 缺的那一塊補齊（self-hosted runner + `docker run`）。
2. Push 上去，在 GitHub Actions 看到綠勾勾就代表部署成功。
3. 打已部署服務的 `/crash` endpoint。
4. 等告警觸發，到 Discord 頻道確認有收到通知。

## Repo 結構（骨架）

```
final-exercise/
├── README.md               # 英文版
├── README.zh-TW.md         # 本檔案
├── app/                    # 服務 source + Dockerfile（待補）
├── .github/workflows/      # CI 完整、CD 骨架（待補）
├── prometheus/             # Scrape 設定 + alert rule（待補）
└── docker-compose.yml      # 本地環境：app + Prometheus（待補）
```

實際內容檔案會在工作坊開始前補上。
