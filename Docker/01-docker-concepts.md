# 01 — Docker 核心概念與基本操作

角色設定：

- **Ocean** — SDC SRE Team 新進成員，負責維護社團的基礎設施和部署流程，還在摸索中，常常搞出各種狀況。
- **Andrew** — 整天在喝酒的 SDC 社長，熟悉社團大小事。
- **Snow** — SDC 開發部長，明明很聰明但常常把事情丟給 Ocean 做，口頭禪是「歐不」。

## 1.1 什麼是容器化？

容器化（Containerization）是一種將「應用程式」及其「完整執行環境」包括程式語言、相依套件、設定檔與環境變數，封裝為方便轉移的元件的技術，而這個元件稱為容器（Container）。無論把容器部署到開發者的筆電、測試伺服器，還是雲端正式環境，執行結果都會保持一致。

這帶來幾個明顯的好處：團隊成員不再需要各自手動建置環境、不同專案的相依套件可以完全隔離、開發與正式環境的差異大幅縮小。容器化特別適合用在多人協作開發、CI/CD 流水線，以及需要快速複製環境的場景。

### 「ㄍㄢˋ，這在我的電腦上明明跑得起來。」

想像一個情境，SDC 有很多個新鮮的肝正在協作開發一個由 Go 撰寫的網頁應用，專案依賴 PostgreSQL 資料庫與特定版本的函式庫。今天，身為 Project Team 的 Andrew 想要請 SRE Team 的 Ocean 把這個服務上線，兩個人的電腦環境差異很大：

| 開發者 | 作業系統 | Go 版本 | PostgreSQL 版本 | 函式庫版本 |
|--------|---------|---------|----------------|-----------|
| Andrew | macOS   | 1.21    | 15             | v2.3.1    |
| Ocean    | Ubuntu  | 1.19    | 14             | v2.1.0    |

Andrew 的程式原本在 macOS 上跑得順順的，但當程式轉移到 Ocean 的電腦之後就看到終端機噴錯了。Ocean 看著空空如也的 README 不知道要從哪裡改設定，在跟 Claude Code 奮戰兩個小時之後大喝了兩口 Whiskey，一不小心睡著了。

Andrew 因為答應了計中明天要上線，只好自己登入到系辦的機器上部署服務。因為正式環境又是另一套設定，程式剛一執行就跑出了 2000 行的 errors。可憐的 Andrew 只好一邊咒罵 SRE Team，一邊喝著自己的 Triple espresso with Vodka，跟這些 errors 奮鬥到天亮。

還好這些不是真實故事，~~因為 Ocean 會乖乖地部署完服務再睡覺~~，聰明的 Docker 也會避免這件事情發生。

### 所以 Docker 如何解決這個問題？？？

開發者將執行環境的完整規格撰寫為 Dockerfile，定義以下資訊：

- 使用哪個版本的 Go
- 需要安裝哪些套件
- 需要哪些環境變數
- 程式如何啟動

任何人在任何機器上，都能透過 Docker 依照 Dockerfile 自動建置出完全相同的容器。新成員加入專案不再需要逐步照著手冊設定環境，執行一個指令即可開始工作。

### 容器化的適用場景

- 多人協作開發：不同人使用不同作業系統與不同版本的工具，容器確保每個人面對完全相同的執行環境。

- 解決相依性衝突：不同專案可能依賴同一套件的不同版本。容器讓每個專案在隔離的空間中獨立運作，彼此不受干擾。

- 標準化的專案啟動程序：Dockerfile 是可執行的環境文件，明確記錄專案需求與啟動方式，取代容易過時的手動安裝說明。

- 一致的部署流程：開發、測試、正式環境使用同一個設定流程，大幅降低環境差異導致的問題。

- 快速實驗：想試 PostgreSQL 17 或 Redis 新版本？一行指令就可以快速啟動，並且用完直接回收，不需要安裝也不會汙染系統環境。

> 容器化將「環境」從各自的機器中抽離，轉變為可版本控制、可共享、可重現的成品。

## 1.2 容器 vs 虛擬機器

Ocean 在系上修過作業系統，對虛擬機器（VM）不陌生，計中機房裡就有好幾台 VM 在跑各種服務。在聽完 Snow 對 Docker 的介紹之後，Ocean 的第一個問題是：「這跟 VM 有什麼不一樣？不都是把東西隔離起來跑」

Snow：問得好，容器跟 VM 要解決的問題確實差不多，但做法完全不一樣。

### 虛擬機器的運作方式

虛擬機器透過 Hypervisor（虛擬機管理程式）在硬體上模擬出多台獨立電腦。每台 VM 都有完整的作業系統，從核心到使用者空間全部包在裡面。

你可以把 VM 想成大樓裡的獨立公寓，各自有完整的水電管路跟門牌。隔離性很強，但每一間都要重複建設同樣的基礎設施，資源利用率偏低。

常見的 Hypervisor 有 VMware ESXi、KVM、Microsoft Hyper-V。

### 容器的運作方式

容器不模擬硬體，而是直接利用 Linux 核心本身的功能（Namespace + Cgroup）來隔離程序。所有容器共用同一個宿主機核心，每個容器只包含應用程式跟它需要的函式庫。

容器比較像大樓裡的 co-working space，共用水電、網路、電梯這些基礎設施，各個團隊有自己獨立的工作區域，互不干擾。省空間、啟動快，但隔離強度不如獨立公寓。

### 架構比較

```
┌──────────────────────────────┬──────────────────────────────┐
│        虛擬機器 (VM)          │        容器 (Container)       │
├──────────────────────────────┼──────────────────────────────┤
│                              │                              │
│  ┌────────┐  ┌────────┐      │  ┌────────┐  ┌────────┐      │
│  │ App A  │  │ App B  │      │  │ App A  │  │ App B  │      │
│  ├────────┤  ├────────┤      │  ├────────┤  ├────────┤      │
│  │ Bins/  │  │ Bins/  │      │  │ Bins/  │  │ Bins/  │      │
│  │ Libs   │  │ Libs   │      │  │ Libs   │  │ Libs   │      │
│  ├────────┤  ├────────┤      │  └────┬───┘  └───┬────┘      │
│  │Guest OS│  │Guest OS│      │       │          │           │
│  └────┬───┘  └───┬────┘      │  ┌────┴──────────┴────┐      │
│  ┌────┴──────────┴────┐      │  │   Docker Engine     │     │
│  │    Hypervisor      │      │  ├─────────────────────┤     │
│  ├────────────────────┤      │  │     Host OS         │     │
│  │     Host OS        │      │  ├─────────────────────┤     │
│  ├────────────────────┤      │  │    Infrastructure   │     │
│  │   Infrastructure   │      │  └─────────────────────┘     │
│  └────────────────────┘      │                              │
│                              │                              │
└──────────────────────────────┴──────────────────────────────┘
```

注意右邊，沒有 Guest OS 那一層。這就是容器比 VM 輕這麼多的原因。

| 比較項目 | 虛擬機器 (VM) | 容器 (Container) |
|---------|--------------|-----------------|
| **隔離方式** | 硬體層虛擬化 (Hypervisor) | 作業系統層虛擬化 (Namespace + Cgroup) |
| **Guest OS** | 每台 VM 都有完整的 OS | 共用宿主機核心，無 Guest OS |
| **啟動時間** | 分鐘級（需要開機） | 秒級（啟動一個程序） |
| **映像檔大小** | GB 級（含完整 OS） | MB 級（App + Libs） |
| **資源佔用** | 高（每台都需分配 CPU、記憶體給 OS） | 低（共用核心，按需分配） |
| **隔離強度** | 強（完整硬體隔離） | 較弱（共用核心，靠 Namespace 隔離） |
| **部署密度** | 一台伺服器通常跑數台至數十台 VM | 一台伺服器可跑數十至數百個容器 |
| **適用場景** | 強隔離需求、不同 OS、遺留系統 | 微服務、CI/CD、快速部署、開發環境 |

### 選擇指引

實際上，這兩個東西經常搭配使用。像是在雲端平台上開一台 VM（EC2 / GCE），然後在 VM 裡面跑 Docker 容器。

**適合使用 VM 的場景：**
- 需要執行不同的作業系統（例如在 Linux 上執行 Windows）
- 需要硬體等級的隔離（多租戶環境、金融法規要求）
- 遺留系統無法容器化

**適合使用容器的場景：**
- CI/CD 流水線
- 開發與測試環境
- 需要快速橫向擴展（scale out）的應用

容器底層靠的是 Linux 核心的 Namespace、Cgroup、Union Filesystem 三個機制來實現隔離。有興趣深入了解的話可以看 [附錄 D](appendix.md#附錄-d容器底層技術)。

---

## 1.3 第一次跑 Docker

Ocean 聽完概念坐不住了，打開終端機想要親手試試看。幸好每次 Andrew 都會趁 Ocean 不在的時候，偷偷往他電腦裡面下載一些怪東西，而他前段時間下載的 OrbStack 剛好可以用來啟動 Docker。

安裝 docker:

```bash
brew install orbstack
```

因為 Andrew 歧視 Windows 系統，所以 Windows 用戶只好請你問你的 AI 夥伴怎麼裝 Docker 了。

```bash
# 驗證安裝
docker version

# 查看 Docker 系統資訊
docker info

# 執行測試容器
docker run hello-world
```

**預期輸出：**

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

看到這段訊息就代表 Docker 裝好了。
接下來 Ocean 打算試點更有感覺的，跑一個真正的網頁伺服器：

```bash
# 跑一個 Nginx 網頁伺服器
docker run --name my-nginx -p 8080:80 nginx:1.27-alpine
```

一行指令就把一個網頁伺服器跑起來了。原來，`docker run` 是運行一個容器的意思，後面接了一些參數。`-p 8080:80` 把容器的 80 port 對映到你電腦的 8080 port，所以我們可以在 `http://localhost:8080` 找到這個應用，看到 404 Not Found 就代表服務啟動起來了。測試完成之後可以清理一下：

```bash
docker stop my-nginx && docker rm my-nginx
```

---

## 1.4 Docker 架構

Ocean 看著瀏覽器上出現的 Nginx 歡迎頁面，進入了自信之巔。但他馬上冒出下一個問題：「剛剛只打了一行指令，背後到底發生什麼事？」此時，Snow 的聲音從遠方冒了出來，而他手上拿著一張 Docker 架構圖，像是已經等著有人問這個問題很久了 ......

```
┌──────────────────────────────────────────────────────────────────┐
│                         Docker 架構                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                                │
│  │ Docker CLI   │                                                │
│  │              │                                                │
│  │ docker run   │                                                │
│  │ docker build │                                                │
│  │ docker pull  │                                                │
│  └──────┬───────┘                                                │
│         │  REST API（通常透過 Unix Socket）                        │
│         ▼                                                        │
│  ┌──────────────────────────────────────────┐                    │
│  │          Docker Daemon                   │                    │
│  │                                          │                    │
│  │  接收 Client 指令，協調所有 Docker 操作     │                    │
│  │                                          │                    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐  │                    │
│  │  │ Image    │ │ Network  │ │ Volume   │  │                    │
│  │  │ 管理      │ │ 管理     │ │ 管理      │  │                    │
│  │  └──────────┘ └──────────┘ └──────────┘  │                    │
│  └──────────────────┬───────────────────────┘                    │
│                     │                                            │
│                     ▼                                            │
│  ┌──────────────────────────────────────────┐                    │
│  │            containerd                    │                    │
│  │                                          │                    │
│  │          管理容器生命週期                  │                    │
│  └──────────────────┬───────────────────────┘                    │
│                     │                                            │
│                     ▲                                            │
│                     │ 拉取映像檔                                  │
│                     ▼                                            │
│  ┌──────────────────────────────────────────┐                    │
│  │          Docker Registry                 │                    │
│  │                                          │                    │
│  │  存放和分發映像檔的倉庫，例如                │                    │
│  │  nginx, postgres, redis, golang, ubuntu  │                    │
│  └──────────────────────────────────────────┘                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Docker Client

就是你在終端機用的 `docker` 命令列工具。它本身不管理容器，只負責把你的指令轉成 REST API 請求丟給 Docker Daemon 處理。Client 跟 Daemon 預設透過 Unix Socket（`/var/run/docker.sock`）溝通，通常在同一台機器上，但也可以透過 TCP 連到遠端的 Daemon。

```bash
# 查看 Docker Client 與 Daemon 的版本資訊
docker version
```

### Docker Daemon

Docker Daemon（程序名稱為 `dockerd`）是 Docker 的核心服務，一直在背景執行。它接收 Client 丟過來的 API 請求，負責「協調」映像檔管理、容器建立、網路設定跟儲存管理這些操作。

舉個例子，你跑 `docker run nginx` 的時候，Daemon 會依序做這些事：
1. 檢查本機是否有 `nginx` 映像檔
2. 若無，從 Registry 下載
3. 請 containerd 建立並啟動容器
4. 設定網路（分配 IP、建立 bridge）
5. 掛載 Volume
6. 回報結果給 Client

dockerd 是領導者（aka AK），他負責規劃與安排任務，不負責執行。實際的運作會交給 containerd 與更底層的工人去執行 Linux 指令。

### containerd（容器執行時期）

containerd 是負責管理容器生命週期的高階執行環境（aka 高級主管）。包括拉取跟推送映像檔、建立和刪除容器、管理映像檔儲存。

**Docker Registry（映像檔倉庫）**

存放跟分發映像檔的服務，你可以把它想成映像檔的 GitHub。

| Registry | 說明 |
|----------|------------------|
| **Docker Hub** | 官方公開倉庫，有大量社群映像檔（預設） |
| **GitHub Container Registry (ghcr.io)** | GitHub 提供的映像檔倉庫 |
| **Harbor** | 開源的自建映像檔倉庫，SDC 的映像檔管理是用這個。 |

### 一個指令的完整旅程

來看看當你打下 `docker run nginx` 之後，背後到底發生了什麼事：

```
┌─────────────────────────────────────────────────────────────┐
│  docker run nginx 的完整流程                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 使用者輸入 docker run nginx                               │
│     │                                                       │
│     ▼                                                       │
│  2. Docker CLI 將指令轉換為 REST API 請求                      │
│     POST /containers/create + POST /containers/{id}/start   │
│     │                                                       │
│     ▼                                                       │
│  3. Docker Daemon 收到請求                                   │
│     ├─→ 本機有 nginx 映像檔嗎？                               │
│     │   ├─ 有 → 直接使用                                     │
│     │   └─ 沒有 → 從 Registry 下載                           │
│     │                                                       │
│     ▼                                                       │
│  4. Daemon 請 containerd 建立容器                            │
│     │                                                       │
│     ▼                                                       │
│  5. 容器啟動完成，nginx 開始運作                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

> **macOS 跟 Windows 的使用者注意**：Docker 容器需要 Linux 核心才能跑。在 macOS 跟 Windows 上，Docker Desktop 會在背景偷偷開一台輕量的 Linux VM，Docker Daemon 其實是跑在這台隱藏的 VM 裡面。

---

## 1.5 映像檔與容器的關係

回想前面學到的東西，Ocean 知道了自己的服務只要跑在容器裡面，就可以容易的轉移到別人的電腦上面。但前面幾節一直出現「映像檔」這個詞，讓他腦袋有點混亂。Ocean 舉手問 Snow：「映像檔跟容器不是同一個東西嗎？」Snow 笑了一下：「很多人一開始都這樣以為，但它們其實完全不同。」

映像檔（Image）是一個打包好的、唯讀的檔案，裡面定義並包含了運行一個應用程式所需的完整執行環境：作業系統的基礎檔案、程式語言的執行環境、你的程式碼、相依套件、設定檔。當你用 `docker run` 把映像檔跑起來，跑起來的那個東西就叫做容器（Container）。映像檔是一個不會變的藍圖，但你可以從同一個映像檔跑出很多個容器，每個容器各自獨立運作。

剛才跑 `docker run hello-world` 的時候，終端機上先後出現了 `Unable to find image 'hello-world:latest' locally`，以及 `Pulling from library/hello-world` 的訊息。他代表我們本地沒有那個藍圖，所以自動往 registry （預設是 Docker Hub）pull 映像檔，並繼續啟動容器。


### 概念對比

| 比喻 | 映像檔（Image） | 容器（Container） |
|------|----------------|-------------------|
| **建築** | 建築藍圖 | 蓋好的房子 |
| **程式設計** | Class（類別定義） | Instance（實體物件） |
| **作業系統** | ISO 映像檔 | 執行中的系統 |

### 映像檔的分層架構

映像檔不是一個大檔案，而是由好幾個唯讀的 Layer 疊起來的。以 `nginx:1.27-alpine` 為例，你可以用 `docker history` 看到它的分層：

```bash
docker history nginx:1.27-alpine
```

每一層都記錄了一個變更：最底層是 Alpine Linux 的基礎檔案，往上可能是安裝 Nginx、複製設定檔、設定啟動指令等等。這些層全部疊在一起，就組成了完整的映像檔。這個分層設計有幾個好處：

- 磁碟共用：假設 10 個映像檔都用 `alpine:3.21` 當基底，Alpine 那一層在磁碟上只需要存一份，省下空間。

- 傳輸效率：拉取映像檔的時候，只需要下載你本機還沒有的 Layer。如果你已經有 Alpine 的基礎層，拉 `nginx:1.27-alpine` 這張 Image（nginx built on Alpine layer），就只需要下載 Nginx 相關的層。

### 容器的可寫層

映像檔的每一層都是唯讀的，不能改。那容器跑起來之後產生的資料寫在哪？

Docker 會在映像檔的所有唯讀層上面加一層可讀寫的容器層。容器執行期間的所有變更（新建檔案、修改設定、寫入日誌）都存在這一層。

```
┌─────────────────────────────────────────────┐
│  Container = Image Layers + 可寫層           │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────────────────────────────┐       │
│  │  Container Layer（可讀寫）         │      │
│  │  新建的檔案、修改的設定、日誌等       │      │
│  │                                  │       │
│  │  ⚠ 容器刪除時，這一層就消失了！      │       │
│  ├──────────────────────────────────┤       │
│  │  Layer 3: Nginx 啟動設定（唯讀）   │       │
│  ├──────────────────────────────────┤       │
│  │  Layer 2: 安裝 Nginx（唯讀）       │Image  │
│  ├──────────────────────────────────┤Layers │
│  │  Layer 1: Alpine Linux（唯讀）    │       │
│  └──────────────────────────────────┘       │
│                                             │
└─────────────────────────────────────────────┘
```

### 多容器共用映像檔

```
┌────────────────────────────────────────────────────────────┐
│                 多容器共用同一個映像檔                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Container A          Container B          Container C     │
│  ┌──────────┐        ┌──────────┐        ┌──────────┐      │
│  │ 可寫層 A  │        │ 可寫層 B  │        │ 可寫層 C  │      │
│  │ (日誌、   │        │ (不同的   │        │ (各自獨   │      │
│  │  暫存等)  │        │  資料)    │        │  立的)    │     │
│  └─────┬────┘        └─────┬────┘        └─────┬────┘      │
│        │                   │                   │           │
│        └───────────────────┼───────────────────┘           │
│                            ▼                               │
│              ┌──────────────────────┐                      │
│              │  共用的映像檔 Layer    │                      │
│              │  (唯讀，只存一份)      │                      │
│              │                      │                      │
│              │  Layer 3: Nginx config│                      │
│              │  Layer 2: Nginx      │                      │
│              │  Layer 1: Alpine     │                      │
│              └──────────────────────┘                      │
│                                                            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 映像檔命名與 Tag

每個映像檔可以有多個 Tag 來區分版本。命名格式長這樣：

```
┌────────────────────────────────────────────────────────────┐
│                    映像檔命名格式                            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  完整格式：                                                  │
│  [registry/][username/]repository[:tag]                    │
│                                                            │
│  範例：                                                     │
│  docker.io/library/nginx:1.27-alpine                       │
│  ├────────┘ ├─────┘ ├───┘ ├─────────┘                      │
│  │          │       │     └─ Tag（版本標記）                 │
│  │          │       └─ Repository（映像檔名稱）              │
│  │          └─ Username（官方映像檔為 library，通常省略）      │
│  └─ Registry（預設 docker.io，通常省略）                      │
│                                                            │
│  常見 Tag 慣例：                                             │
│  nginx:latest        → 最新版（不建議正式環境使用）             │
│  nginx:1.27          → 主要版本                              │
│  nginx:1.27.3        → 精確版本（正式環境建議使用）             │
│  nginx:1.27-alpine   → 基於 Alpine 的精簡版                  │
│  nginx:1.27-bookworm → 基於 Debian Bookworm 的版本           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 概念總結

| 概念 | 映像檔（Image） | 容器（Container） |
|------|----------------|-------------------|
| **性質** | 唯讀模板，由多個 Layer 組成 | 可讀寫的執行實體，映像檔 + 可寫層 |
| **生命週期** | 建置後不可變（Immutable） | 可建立、啟動、暫停、停止、刪除 |
| **儲存** | 分層結構，可跨映像檔共用 | 可寫層存放執行期間的變更 |
| **可重複性** | 同一 Dockerfile 永遠建出相同的映像檔 | 每個容器的可寫層內容可能不同 |
| **數量關係** | 一個映像檔可產生多個容器 | 一個容器只對應一個映像檔 |
| **資料持久性** | 除非主動刪除，否則永久存在 | 可寫層隨容器刪除而消失（需 Volume 持久化） |

---

## 1.6 映像檔基本操作

知道映像檔是什麼之後，來看看怎麼找、下載、管理這些映像檔。

### 搜尋映像檔

```bash
# 從 Docker Hub 搜尋映像檔
docker search nginx

# 建議直接前往 Docker Hub 網站搜尋，資訊更完整
# https://hub.docker.com
```

### 下載映像檔

```bash
# 下載最新版（預設是 :latest）
docker pull nginx

# 下載特定版本
docker pull nginx:1.27

# 下載特定平台版本
docker pull --platform linux/amd64 nginx:1.27

# 下載 Alpine 精簡版（體積更小，適合正式環境）
docker pull nginx:1.27-alpine
```

> **注意**：Tag 就是映像檔的版本標記。正式環境一定要指定明確版本號，例如 `nginx:1.27.3`，否則當軟體升級時，容易出現不相容的問題。


### 列出本機映像檔

```bash
docker images

# 輸出範例：
# REPOSITORY   TAG           IMAGE ID       CREATED       SIZE
# nginx        1.27-alpine   a2bd6dc6e5e6   2 weeks ago   43.3MB
# nginx        1.27          39286ab8a5e1   2 weeks ago   192MB
# hello-world  latest        d2c94e258dcb   9 months ago  13.3kB
```

`nginx:1.27-alpine`（43MB）跟 `nginx:1.27`（192MB）差了快五倍，這就是用 Alpine 當基底的好處。

### 刪除映像檔

```bash
# 刪除指定映像檔
docker rmi nginx:1.27

# 刪除所有未使用的映像檔（dangling images）
docker image prune
```

### 查看映像檔資訊

```bash
# 查看映像檔詳細資訊（層數、環境變數等）
docker inspect nginx:1.27-alpine

# 查看映像檔的建置歷史（每一層的指令）
docker history nginx:1.27-alpine
```

---

## 1.7 容器基本操作

有了映像檔，接下來就是把它跑起來。這一節整理容器從建立到刪除的完整操作。

### 建立並啟動容器

```bash
# 基本語法
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# 前景模式執行（Ctrl+C 可停止）
docker run nginx:1.27-alpine

# 指定容器名稱
docker run --name my-nginx nginx:1.27-alpine

# 執行後自動刪除容器（適合一次性任務，後面可以直接接參數是要執行的命令，nginx -v 是看 nginx 的版本。）
docker run --rm nginx:1.27-alpine nginx -v
```

### 常用 `docker run` 參數

| 參數 | 說明 | 範例 |
|------|------|------|
| `-d` | 背景執行（detach） | `docker run -d nginx` |
| `--name` | 命名容器 | `docker run --name web nginx` |
| `-p` | Port mapping（host:container） | `docker run -p 8080:80 nginx` |
| `-v` | 掛載 Volume | `docker run -v /data:/app/data nginx` |
| `-e` | 設定環境變數 | `docker run -e DB_HOST=db nginx` |
| `--rm` | 停止後自動刪除 | `docker run --rm nginx` |
| `--network` | 指定網路 | `docker run --network=mynet nginx` |

### 列出容器

```bash
# 列出執行中的容器
docker ps

# 列出所有容器(包含已停止的)
docker ps -a

# 只顯示容器 ID
docker ps -q

# 輸出範例：
# CONTAINER ID   IMAGE               STATUS          PORTS                  NAMES
# a1b2c3d4e5f6   nginx:1.27-alpine   Up 10 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

### 容器生命週期管理

```bash
# 停止容器（發送 SIGTERM，等待容器優雅關閉）
docker stop my-nginx

# 啟動已停止的容器
docker start my-nginx

# 強制停止容器（發送 SIGKILL，不建議使用）
docker kill my-nginx

# 刪除已停止的容器
docker rm my-nginx

# 強制刪除執行中的容器
docker rm -f my-nginx

# 刪除所有已停止的容器
docker container prune
```

**容器生命週期圖：**

```
┌─────────────────────────────────────────────────────────────┐
│                    容器生命週期                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  docker create         docker start         docker stop     │
│  ┌──────────┐  ───→   ┌──────────┐  ───→   ┌──────────┐     │
│  │ Created  │         │ Running  │         │ Stopped  │     │
│  └──────────┘  ←───   └──────────┘  ←───   └──────────┘     │
│                                                             │
│  docker run = docker create + docker start                  │
│                                                             │
│              docker rm                                      │
│  任何狀態 ────────────────→ 刪除                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.8 Port Mapping

Ocean 想 demo 給 Andrew 看自己學會用 Docker 了，很帥氣地打了 `docker run -d nginx:1.27-alpine`，打開瀏覽器想要連接 `http://localhost:3000`，結果發現 "This site can’t be reached"。Ocean 以為是容器沒有順利啟動，打了 `docker ps` 卻發現容器跑得好好的。

Andrew 看了一眼指令：「你沒加 `-p`，port 根本沒開出來啊。」

Ocean：「port 是什麼？」Snow：「6」

### Port 是什麼？

一台電腦上同時會跑很多網路服務：Web Server、資料庫、SSH 等。它們共用同一個 IP 位址，作業系統透過 **Port**（連接埠，範圍 0–65535）區分流量該送給哪個程式。

每個容器都有獨立的網路空間（Network Namespace），容器內部的服務對宿主機而言是不可見的。**Port Mapping** 的作用就是在宿主機與容器之間建立一條轉發通道，將宿主機指定連接埠收到的流量轉發至容器內的連接埠。

| 名詞 | 說明 | 範例 |
|------|------|------|
| **Container Port** | 應用程式在容器內監聽的連接埠，僅存在於容器的網路空間中，宿主機無法直接存取 | Node.js `app.listen(3000)` → Container Port 為 3000 |
| **Host Port** | 宿主機對外開放的連接埠，Docker 監聽此埠並轉發流量至對應的 Container Port | `-p 8080:3000` → Host Port 為 8080 |

```
┌─────────────────────────────────────────────────┐
│                    Port Mapping                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  宿主機 (Host)                                   │
│  ┌─────────────────────────────────────┐        │
│  │                                     │        │
│  │   瀏覽器 → http://localhost:8080     │        │
│  │                    │                │        │
│  │                    ▼                │        │
│  │          Host Port 8080             │        │
│  │                    │                │        │
│  │         ┌──────────┼──────────┐     │        │
│  │         │ Container│          │     │        │
│  │         │          ▼          │     │        │
│  │         │  Container Port 80  │     │        │
│  │         │          │          │     │        │
│  │         │     ┌────▼────┐     │     │        │
│  │         │     │  Nginx  │     │     │        │
│  │         │     └─────────┘     │     │        │
│  │         └─────────────────────┘     │        │
│  │                                     │        │
│  └─────────────────────────────────────┘        │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 語法

```bash
# -p <Host Port>:<Container Port>
docker run -d --name web -p 8080:80 nginx:1.27-alpine

# 測試
curl http://localhost:8080
```

### 常用變化

| 用法 | 指令 | 說明 |
|------|------|------|
| 對映多個連接埠 | `docker run -d -p 8080:80 -p 8443:443 nginx` | 同時開放 HTTP 與 HTTPS |
| 限定綁定 IP | `docker run -d -p 127.0.0.1:8080:80 nginx` | 只允許本機連線，不對外開放 |
| 隨機分配 Host Port | `docker run -d -P nginx` | Docker 自動選一個可用的 port，用 `docker port` 查看 |
| 查看對映 | `docker port web` | 列出指定容器的所有 port mapping |

> **注意**：Container Port 不會衝突。每個容器有獨立的網路空間，十個容器都跑 port 3000 完全沒問題，只要 Host Port 不重複就好。

---

## 1.9 Volume

Ocean 學會 Port Mapping 之後，原本就已經在自信之巔的他再度信心大增，決定把社團的 PostgreSQL 資料庫也用 Docker 跑起來。他花了一個下午匯入資料、調整設定，一切運作良好。隔天早上 Ocean 發現容器不知道為什麼停了，他想說沒關係，砍掉重跑一個就好：

```bash
docker rm db && docker run -d --name db postgres:17-alpine
```

打開資料庫一看，裡面比新的還乾淨，喔不對他就是新的。Ocean 崩潰地跑去找 Snow：「我的資料呢？！」Snow 淡定地喝了一口咖啡：「容器砍掉，裡面的可寫層就沒了啊。還記得 1.5 節講的嗎？」

Ocean 這才想起來，容器的可寫層是跟著容器走的，`docker rm` 的瞬間，資料庫的紀錄、使用者上傳的檔案、辛辛苦苦產生的 log，全部歸零。

Snow：「噢不。」

Docker 的 **Volume** 把宿主機的目錄或 Docker 管理的儲存空間掛載到容器裡面，讓資料可以跨容器生命週期保留下來。

常見的掛載方式有兩種：

- **Bind Mount（綁定掛載）**：把宿主機的檔案目錄直接掛進容器。只需在 `docker run` 時添加 `-v /host/path:/container/path` 即為設定完成。因為改完本機的檔案後會直接影響容器內部，適合在開發時同步原始碼。
- **Named Volume（具名資料卷）**：由 Docker 管理的儲存空間，存放在 host 的 `/var/lib/docker/volumes/` 底下。只需在 `docker run` 時添加 `-v mydata:/container/path` 即為設定完成。適合資料庫這種需要持久化的資料，以及不需要從 host 直接存取內容的情況。


```bash
# Bind Mount
docker run -d --name web \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:1.27-alpine
# $(pwd) 在 Linux 當中是當前目錄的意思。

```

```bash
# Named Volume

docker volume create pgdata
# 在使用 Named Volume 時，你需要先像是變數宣告一樣 create 它。

docker run -d --name db \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17-alpine
```

```bash
# 列出所有 Volume
docker volume ls

# 查看 Volume 詳細資訊
docker volume inspect pgdata

# 刪除 Volume
docker volume rm pgdata

# 刪除未使用的 Volume，執行前請先問你家老大。
docker volume prune
```

> **注意**：Bind Mount 要用絕對路徑（不能用相對路徑），Named Volume 只要寫名稱就好。如果路徑是 `/` 開頭，Docker 就會當成 Bind Mount。

---

## 1.10 容器除錯

### 查看日誌

```bash
docker logs my-nginx
```

| 參數 | 效果 | 範例 |
|------|------|------|
| `-f` | 持續追蹤，類似 `tail -f` | `docker logs -f my-nginx` |
| `--tail N` | 只看最後 N 行 | `docker logs --tail 100 my-nginx` |
| `-t` | 顯示時間戳記 | `docker logs -t my-nginx` |
| `--since` | 指定時間之後的日誌 | `docker logs --since 2024-01-01 my-nginx` |

參數可以組合使用，例如 `docker logs -f --tail 50 -t my-nginx` 就是持續追蹤最後 50 行並顯示時間。

### 進入容器內部

```bash
# 在容器內開啟 shell (可以用 sh/bash)
docker exec -it my-nginx sh
# -i: 保持標準輸入開啟（interactive）
# -t: 分配偽終端（tty），兩個一起用是常見操作。

# 執行單一命令（不進入互動模式）
docker exec my-nginx cat /etc/nginx/nginx.conf

# 以 root 身份進入
docker exec -it -u root my-nginx sh
```

### 查看資源使用狀況

```bash
# 即時顯示所有容器的資源使用
docker stats

# 只看特定容器
docker stats my-nginx

# 輸出範例：
# CONTAINER ID   NAME       CPU %   MEM USAGE / LIMIT     MEM %   NET I/O
# a1b2c3d4e5f6   my-nginx   0.00%   3.441MiB / 7.667GiB   0.04%   1.45kB / 0B
```

---

## 1.11 練習 1：執行你的第一個容器

👉 練習步驟請參考 [exercises/01-first-container.md](exercises/01-first-container.md)

完成練習後，Ocean 回頭看看自己在 Part 1 學了什麼：知道容器化是怎麼回事、容器跟 VM 差在哪、映像檔跟容器的關係、怎麼用 `docker run` 把服務跑起來、用 `-p` 開 port、用 `-v` 掛資料、容器壞了怎麼查 log 跟進去除錯。

Snow 看著 Ocean 的練習成果，點了點頭：「不錯嘛，基本操作都會了。剛好 SDC 缺了一個 SRE Team，你要不要來做做看？」

Ocean：「......那要做什麼？」

Snow：「用 Docker Compose 把這些服務的設定步驟寫成檔案。」

→ 下一章：[02-docker-compose.md](02-docker-compose.md)
