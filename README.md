<div align="center">

  <img src="asset/gbtw.jpeg" alt="GBTW Promo Watchdog" width="160" />

  <h1>🛰️ GBTW Promo Watchdog</h1>

  <p>
    <strong>Automated Tactical Intel & Promo Code Radar for <em>Gunship Battle: Total Warfare</em></strong>
  </p>

  <p>
    <img src="https://img.shields.io/badge/Go-1.22+-00ADD8?style=for-the-badge&logo=go&logoColor=white" alt="Go 1.22+" />
    <img src="https://img.shields.io/badge/SQLite-Zero--CGO-003B57?style=for-the-badge&logo=sqlite&logoColor=white" alt="SQLite" />
    <img src="https://img.shields.io/badge/Telegram-Bot_API-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram Bot" />
    <img src="https://img.shields.io/badge/Linux-Systemd-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux Systemd" />
    <img src="https://img.shields.io/badge/RAM_Footprint-~15_MB-ef4444?style=for-the-badge" alt="Low RAM" />
    <img src="https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge" alt="MIT License" />
  </p>

  <p>
    A zero-bloat Go scraper and Telegram alert daemon that queries the official
    <a href="https://gw.joycity.com/en/boards/event">JOYCITY Event Board</a> API for new promo/coupon drops,
    parses the reward breakdown and UTC/local expiration deadline, and pushes one-tap copyable Telegram alerts directly to your phone.
  </p>

</div>

---

## 🎯 Why This Exists

_Gunship Battle: Total Warfare_ surprise coupons often expire within **48 to 72 hours**. Built specifically for resource-constrained VPS environments (such as **Oracle Cloud Always Free 1 GB** instances), this tool runs as a `systemd` oneshot timer—consuming **~15 MB RAM** for ~1.5 seconds per scan and **0 MB** while idle.

## ✨ Key Capabilities

| Feature                              | Description                                                                                                                                      |
| :----------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| 🤝 **Two-Way Telegram Handshake**    | Sends an initialization greeting on first boot and waits for your reply (`confirm` / `hi`) before unlocking the archive backfill and live radar. |
| 📜 **60-Day Chronological Backfill** | Upon handshake confirmation, delivers every promo code from the last 60 days in chronological order, then switches to live 15-minute monitoring. |
| 🔍 **Direct Board JSON Recon**       | Queries JOYCITY's `/en/boards/event/data` endpoint directly—no headless Chromium or Puppeteer overhead.                                          |
| 📋 **One-Tap Copyable Codes**        | Wraps every coupon code in a Telegram `<code>` block so a single tap on mobile copies only the code.                                             |
| 📦 **Zero-CGO Static Binary**        | Powered by `modernc.org/sqlite` for single-binary cross-compilation across `linux/amd64` and `linux/arm64`.                                      |

---

## ⚙️ System Architecture

```text
┌───────────────────────────┐    POST /data       ┌──────────────────────────────┐
│  Systemd Timer (15m)      │ ──────────────────► │  JOYCITY Event Board API     │
│  gbtw-watchdog.timer      │                     │  [gw.joycity.com/en/boards](https://gw.joycity.com/en/boards)    │
└─────────────┬─────────────┘                     └──────────────┬───────────────┘
              │                                                  │
              ▼                                                  ▼
┌───────────────────────────┐    New Post ID?     ┌──────────────────────────────┐
│  SQLite State & Dedup     │ ◄────────────────── │  Detail HTML Parser          │
│  data/seen_coupons.db     │                     │  Code • Rewards • UTC/Local  │
└─────────────┬─────────────┘                     └──────────────┬───────────────┘
              │ Handshake Verified & Unseen                      │
              └─────────────────────────┬────────────────────────┘
                                        ▼
                        ┌───────────────────────────────┐
                        │  Telegram Bot API             │
                        │  sendMessage / getUpdates     │
                        └───────────────────────────────┘
```

---

## 🚀 Quick Start

### 1️⃣ Create a Telegram Bot & Get Your Chat ID

1. Open Telegram, message **`@BotFather`**, send `/newbot`, and copy your **`TELEGRAM_BOT_TOKEN`**.
2. Open a chat with your new bot and send `/start` (or `hi`).
3. Retrieve your numeric **`TELEGRAM_CHAT_ID`** via `@userinfobot` or by running:
   ```bash
   curl -s "[https://api.telegram.org/bot](https://api.telegram.org/bot)<YOUR_BOT_TOKEN>/getUpdates"
   ```

---

### 2️⃣ Clone & Configure

```bash
git clone [https://github.com/MarioOckersz/gbtw-promo-watchdog.git](https://github.com/MarioOckersz/gbtw-promo-watchdog.git)
cd gbtw-promo-watchdog

cp .env.example .env
nano .env
```

Set `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, and `DISPLAY_TIMEZONE` (e.g., `Asia/Colombo`, `America/New_York`, `UTC`).

---

### 3️⃣ Build & Verify Handshake

Requires **Go 1.22+**:

```bash
go mod tidy
go build -ldflags="-s -w" -o bin/watchdog ./cmd/watchdog

# Run 1: Sends initialization greeting and awaits your reply
./bin/watchdog

# -> Reply "confirm" or "hi" to your bot in Telegram <-

# Run 2: Confirms handshake, delivers last 60 days of codes, and arms live radar
./bin/watchdog
```

---

### 4️⃣ Deploy as a Systemd Timer (Linux VPS)

Automate scans every 15 minutes with **0 MB idle RAM**:

```bash
sudo cp deploy/gbtw-watchdog.service /etc/systemd/system/
sudo cp deploy/gbtw-watchdog.timer /etc/systemd/system/

# Update WorkingDirectory and User in gbtw-watchdog.service if not using /opt/gbtw-promo-watchdog
sudo systemctl daemon-reload
sudo systemctl enable --now gbtw-watchdog.timer
```

#### 🛠️ Useful Operations Commands

```bash
# Check next scheduled scan
systemctl list-timers gbtw-watchdog.timer

# View recent execution logs
journalctl -u gbtw-watchdog.service -n 50

# Trigger an immediate manual scan
sudo systemctl start gbtw-watchdog.service
```

---

## ⚠️ Disclaimer

This project is an independent open-source community utility and is not affiliated with, endorsed by, or sponsored by **JOYCITY Corporation**. All game trademarks and assets belong to their respective owners.
