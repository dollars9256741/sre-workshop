# 最後練習：CI/CD + 監控整條串起來

## 課程簡介

25 分鐘的綜合練習，把 Docker、CI/CD、Prometheus 三項技能串在一起。你會透過 GitHub Actions 把服務部署到 SDC 機器，把 metric 吐給 Prometheus，並在服務壞掉時收到 Discord 告警。

> **練習本體在另一個 repo：** [Ocean1029/sre-workshop-capstone](https://github.com/Ocean1029/sre-workshop-capstone)。
>
> 那個 repo 才是你會 clone、改、push 的對象。這個目錄只放給講師跟看 workshop 主 repo 的人看的簡介。

**capstone repo 提供：**

- 服務程式碼與 Dockerfile
- 大部分的 GitHub Actions workflow（CI 完整、CD 留 TODO 給你填）
- Prometheus scrape 設定與告警規則
- Part 1 本機跑的 `docker-compose.yml`

**SDC 機器上已經跑著：**

- GitHub self-hosted runner
- Prometheus 與 Alertmanager（Alertmanager 已經接好 Discord channel）

## 適合誰

- 已經完成今天上半天的 Docker、CI/CD、Prometheus 三個 workshop
- 想把三塊兜起來實際走一遍

## 流程

### Part 0 — 走過一次架構（5 分鐘）

目標架構：repo → GitHub Actions → self-hosted runner → SDC 機器上 `docker run` → Prometheus scrape → Alertmanager → Discord。

### Part 1 — 在本機跑起來（10 分鐘）

1. `docker compose up` 把服務跟 Prometheus 一起跑起來。
2. 確認 Prometheus 抓得到服務的 metric。
3. 打開 Prometheus UI，找服務的 metric。
4. 打 `/crash` endpoint，看 counter 增加。

### Part 2 — 接上 CD，觸發一次真的告警（10 分鐘）

1. 照著 CI/CD workshop ch04 的做法，把缺少的 CD job 補完（self-hosted runner + `docker run`）。
2. 在分到的 `student-<ID>` branch 上 push；到 GitHub Actions 看 pipeline 跑綠。
3. 打部署後的 `/crash`。
4. 等告警觸發，到 Discord 看通知有沒有進來。

## 開始

```bash
git clone https://github.com/Ocean1029/sre-workshop-capstone.git
cd sre-workshop-capstone
docker compose up --build
```

完整步驟在 capstone repo 的 [README](https://github.com/Ocean1029/sre-workshop-capstone#flow)。
