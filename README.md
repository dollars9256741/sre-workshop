# SRE Workshop

**English** · [繁體中文](README.zh-TW.md)

A hands-on workshop series covering essential SRE (Site Reliability Engineering) skills — Docker containerization, GitHub Actions CI/CD automation, and Prometheus monitoring. Caps off with a final exercise that ties all three together.

## Workshops

| Workshop | Duration | Description |
|----------|----------|-------------|
| [Docker](Docker/) | 90 min | Containers, Docker Compose, and Dockerfile |
| [CI/CD](CI-CD/) | 100 min | GitHub Actions, CI pipeline, self-hosted runner deploy |
| [Prometheus](Prometheus/) | 60 min | Metrics collection, alerting, and Discord notifications |
| [Final Exercise](final-exercise/) | 25 min | Deploy via CI/CD, scrape with Prometheus, alert on Discord |

## Prerequisites Checklist

Run through this list before the workshop. Each item has install instructions and a verify command below.

- [ ] Git
- [ ] Go 1.24+
- [ ] Docker (Docker Compose is bundled)
- [ ] make
- [ ] VS Code (with YAML extension)
- [ ] GitHub account
- [ ] Docker Hub account
- [ ] Discord account

**Windows users**: do everything inside **WSL 2**. Install WSL first, then follow the "Windows (WSL)" path for every tool below.
Open PowerShell as administrator to install WSL 2:

```powershell
wsl --install
```
Run the following command to re-enter WSL after restarting PowerShell:
```powershell
wsl
```

### Install & verify

<details>
<summary><b>Git</b></summary>

- **macOS**: `brew install git` (or let Xcode CLI tools install it on first `git` use)
- **Windows (WSL)**: `sudo apt update && sudo apt install -y git`

Verify:
```bash
git --version
# git version 2.x
```

</details>

<details>
<summary><b>Go 1.24+</b></summary>

- **macOS**: `brew install go`
- **Windows (WSL)**: `sudo snap install go --classic` — or download from [go.dev/dl](https://go.dev/dl/)

Verify:
```bash
go version
# go version go1.24.x darwin/arm64
```

</details>

<details>
<summary><b>Docker (includes Docker Compose)</b></summary>

- **macOS**: `brew install --cask orbstack`, then launch OrbStack
- **Windows (WSL)**: install [Docker Desktop](https://www.docker.com/products/docker-desktop/) and enable WSL 2 integration in *Settings → Resources → WSL Integration*. Run `docker` commands from inside your Ubuntu WSL shell.

Verify:
```bash
docker --version
# Docker version 27.x, build ...

docker compose version
# Docker Compose version v2.x
```

</details>

<details>
<summary><b>make</b></summary>

- **macOS**: ships with Xcode CLI tools — run `xcode-select --install` if missing
- **Windows (WSL)**: `sudo apt install -y make`

Verify:
```bash
make --version
# GNU Make 4.x
```

</details>

<details>
<summary><b>VS Code</b></summary>

- **macOS**: `brew install --cask visual-studio-code` — or download from [code.visualstudio.com](https://code.visualstudio.com/)
- **Windows**: download from [code.visualstudio.com](https://code.visualstudio.com/); install the [WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) so you can edit files inside WSL

Then install the **YAML** extension from the Extensions panel inside VS Code.

</details>

<details>
<summary><b>GitHub / Docker Hub / Discord accounts</b></summary>

Sign up (no install needed):

- **GitHub** — [github.com/signup](https://github.com/signup)
- **Docker Hub** — [hub.docker.com/signup](https://hub.docker.com/signup)
- **Discord** — [discord.com/register](https://discord.com/register)

</details>

## Repository Structure

```
sre-workshop/
├── README.md               # This file
├── README.zh-TW.md         # Chinese version
├── Docker/                 # Docker workshop
├── CI-CD/                  # CI/CD workshop
├── Prometheus/             # Prometheus workshop
└── final-exercise/         # End-of-day capstone exercise
```

## License

This material is intended for educational use within SDC workshops.
