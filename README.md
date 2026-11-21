<div align="center">

  <img src="asset/gbtw.jpeg" alt="GBTW Promo Watchdog Emblem" width="150" />

  <h1>🛰️ GBTW Promo Watchdog</h1>

  <p>
    <strong>Automated Tactical Intel & Promo Code Radar for <em>Gunship Battle: Total Warfare</em></strong>
  </p>

  <p>
    <img src="https://img.shields.io/badge/Go-1.22+-00ADD8?style=for-the-badge&logo=go&logoColor=white" alt="Go 1.22+" />
    <img src="https://img.shields.io/badge/SQLite-Zero--CGO-003B57?style=for-the-badge&logo=sqlite&logoColor=white" alt="SQLite" />
    <img src="https://img.shields.io/badge/WhatsApp-Alerts-25D366?style=for-the-badge&logo=whatsapp&logoColor=white" alt="WhatsApp Alerts" />
    <img src="https://img.shields.io/badge/Linux-Systemd-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux Systemd" />
    <img src="https://img.shields.io/badge/RAM_Footprint-~15_MB-ef4444?style=for-the-badge" alt="Low RAM" />
    <img src="https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge" alt="MIT License" />
  </p>

  <p>
    A zero-bloat Go scraper and WhatsApp alert daemon that monitors the official
    <a href="https://gw.joycity.com/en/boards/event">JOYCITY Event Board</a> for new promo/coupon drops,
    parses the exact reward table and expiration deadline, and pushes a tap-to-copy WhatsApp alert directly to your phone.
  </p>

</div>

---

## 🎯 Why This Exists

*Gunship Battle: Total Warfare* surprise coupons often expire within **48 to 72 hours**. Built specifically for resource-constrained VPS environments, this tool runs as a `systemd` oneshot timer—consuming **~15 MB RAM** for ~1.5 seconds per scan and **0 MB** while idle.

## ✨ Key Capabilities

| Feature | Description |
| :--- | :--- |
| 🔍 **Smart Board Recon** | Scans JOYCITY event notices for `Coupon` / `Promo Code` drops and skips already-processed post IDs via SQLite before fetching detail pages. |
| 🎁 **Reward & Expiry Extraction** | Parses HTML reward tables (Gold, 1M Resource Boxes, Medals of Merit, Blueprints) and converts `UTC` deadlines into your configured local timezone. |
| 📋 **Tap-to-Copy WhatsApp Alerts** | Places the coupon code on an isolated line so a single long-press on mobile copies only the code—zero text selection struggle. |
| 🔌 **Dual WhatsApp Backend** | Supports **CallMeBot** (1-step setup) and **Meta WhatsApp Cloud API** (official free tier) behind a clean provider interface. |
| 📦 **Zero-CGO Static Binary** | Powered by `modernc.org/sqlite` for single-binary cross-compilation across `linux/amd64` and `linux/arm64`. |

---

## ⚙️ System Architecture

```text
┌───────────────────────────┐      HTTP GET       ┌──────────────────────────────┐
│  Systemd Timer (30m)      │ ──────────────────► │  JOYCITY GBTW Event Board    │
│  gbtw-watchdog.timer      │                     │  [gw.joycity.com/en/boards](https://gw.joycity.com/en/boards)    │
└─────────────┬─────────────┘                     └──────────────┬───────────────┘
              │                                                  │
              ▼                                                  ▼
┌───────────────────────────┐    New Post ID?     ┌──────────────────────────────┐
│  SQLite Deduplicator      │ ◄────────────────── │  HTML Parser & Sanitizer     │
│  data/seen_coupons.db     │                     │  Code • Rewards • UTC/Local  │
└─────────────┬─────────────┘                     └──────────────┬───────────────┘
              │ Unseen                                           │
              └─────────────────────────┬────────────────────────┘
                                        ▼
                        ┌───────────────────────────────┐
                        │  WhatsApp Dispatch            │
                        │  CallMeBot / Meta Cloud API   │
                        └───────────────────────────────┘
```

---

## 🚀 Quick Start

### 1️⃣ Obtain a WhatsApp API Key

<details open>
<summary><strong>Option A: CallMeBot (Fastest — 60 Seconds)</strong></summary>

1. Save **`+34 623 78 64 49`** to your phone contacts. *(If CallMeBot's shared gateway number is temporarily full, check [callmebot.com](https://www.callmebot.com/blog/free-api-whatsapp-messages/) for the active number or use Option B).*
2. Send this exact WhatsApp message to the contact:
   ```text
   I allow callmebot to send me messages
   ```
3. Copy the `APIKEY` replied by the bot.

</details>

<details>
<summary><strong>Option B: Meta WhatsApp Cloud API (Official Free Tier)</strong></summary>

1. Create a free app at [developers.facebook.com](https://developers.facebook.com/) and add the **WhatsApp** product.
2. Copy your **Access Token** and **Phone Number ID**.

</details>

---

### 2️⃣ Clone & Configure

```bash
git clone [https://github.com/MarioOckersz/gbtw-promo-watchdog.git](https://github.com/MarioOckersz/gbtw-promo-watchdog.git)
cd gbtw-promo-watchdog

cp .env.example .env
nano .env
```

Update `.env` with your target number and credentials:
- `WHATSAPP_PHONE` — International format **without** `+` or spaces (e.g., `15551234567`)
- `CALLMEBOT_API_KEY` — Your CallMeBot key (or set `WHATSAPP_PROVIDER=meta` and fill Meta tokens)
- `DISPLAY_TIMEZONE` — IANA timezone for local expiry conversion (e.g., `Asia/Colombo`, `America/New_York`, `UTC`)

---

### 3️⃣ Build & Test Run

Requires **Go 1.22+**:

```bash
mkdir -p bin data
go build -ldflags="-s -w" -o bin/watchdog ./cmd/watchdog

# Run once manually to verify scraping and WhatsApp delivery
./bin/watchdog
```

---

### 4️⃣ Deploy as a Systemd Timer (Linux VPS)

Automate checks every 30 minutes with **0 MB idle RAM**:

```bash
sudo cp deploy/gbtw-watchdog.service /etc/systemd/system/
sudo cp deploy/gbtw-watchdog.timer /etc/systemd/system/

# Edit WorkingDirectory and User in gbtw-watchdog.service if not using /opt/gbtw-promo-watchdog
sudo systemctl daemon-reload
sudo systemctl enable --now gbtw-watchdog.timer
```

#### 🛠️ Useful Operations Commands

```bash
# Check next scheduled scan
systemctl list-timers gbtw-watchdog.timer

# View recent execution logs
journalctl -u gbtw-watchdog.service -n 50

# Trigger an immediate manual scan via systemd
sudo systemctl start gbtw-watchdog.service
```

---

## 📱 Sample WhatsApp Alert

```text
🚁 GBTW PROMO CODE DETECTED

CHRONICLE10T

🎁 Rewards:
• 1,000 Gold
• 10x 1M Resource Box
• 150x Medals of Merit

⏳ Expires:
• UTC: Nov 30, 2026 14:59 UTC
• Local: Nov 30, 2026 20:29 IST

🔗 [https://gw.joycity.com/en/boards/event/](https://gw.joycity.com/en/boards/event/)...
```

---

## ⚠️ Disclaimer

This project is an independent open-source community utility and is not affiliated with, endorsed by, or sponsored by **JOYCITY Corporation**. All game trademarks and assets belong to their respective owners.
