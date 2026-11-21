# gbtw-promo-watchdog
# GBTW Promo Watchdog

A zero-bloat Go scraper and WhatsApp alert daemon that monitors the official **Gunship Battle: Total Warfare** [JOYCITY Event Board](https://gw.joycity.com/en/boards/event) for new promo/coupon codes, parses the exact reward table and expiration deadline, and pushes a tap-to-copy WhatsApp notification directly to your phone.

Built specifically for resource-constrained VPS environments (such as Oracle Cloud Always Free 1 GB instances): runs as a `systemd` oneshot timer, uses **~15 MB RAM** during execution, and **0 MB** while idle.

## Features
- **Smart Board Filtering:** Scans JOYCITY event notices for coupon/promo announcements and skips already-processed post IDs via SQLite before fetching detail pages.
- **Reward & Expiry Extraction:** Parses HTML reward tables (Gold, Resource Boxes, Medals of Merit, Blueprints) and converts `UTC` expiration deadlines into your configured local timezone.
- **Tap-to-Copy WhatsApp Alerts:** Places the coupon code on an isolated line so a single long-press on mobile copies only the code.
- **Dual WhatsApp Backend:** Supports **CallMeBot** (1-step setup) and **Meta WhatsApp Cloud API** (official free tier).
- **Zero-CGO Static Binary:** Uses `modernc.org/sqlite` for effortless single-binary compilation across `linux/amd64` and `linux/arm64`.

---

## Quick Start

### 1. Get a WhatsApp API Key

#### Option A: CallMeBot (Fastest)
1. Save **`+34 623 78 64 49`** to your phone contacts. *(Note: If CallMeBot's shared bot number is temporarily full, check [callmebot.com](https://www.callmebot.com/blog/free-api-whatsapp-messages/) for the active gateway number or use Option B).*
2. Send the WhatsApp message:
   ```text
   I allow callmebot to send me messages
   ```
  
3. Copy the `APIKEY` replied by the bot.

#### Option B: Meta WhatsApp Cloud API (Official Free Tier)
1. Create a free app at [developers.facebook.com](https://developers.facebook.com/) and add the **WhatsApp** product.
2. Copy your **Temporary/System Access Token** and **Phone Number ID**.

---

### 2. Clone & Configure

```bash
git clone [https://github.com/MarioOckersz/gbtw-promo-watchdog.git](https://github.com/MarioOckersz/gbtw-promo-watchdog.git)
cd gbtw-promo-watchdog

cp .env.example .env
nano .env
```

Fill in `WHATSAPP_PHONE` (e.g., `15551234567` without `+`), your `CALLMEBOT_API_KEY` (or Meta credentials), and your preferred `DISPLAY_TIMEZONE` (e.g., `Asia/Colombo`).

---

### 3. Build & Test Run

Requires Go 1.22+:

```bash
mkdir -p bin data
go build -ldflags="-s -w" -o bin/watchdog ./cmd/watchdog

# Run once manually to verify scraping and WhatsApp delivery
./bin/watchdog
```

---

### 4. Deploy as a Systemd Timer (Linux VPS)

To check for new codes automatically every 30 minutes without keeping a process resident in RAM:

```bash
sudo cp deploy/gbtw-watchdog.service /etc/systemd/system/
sudo cp deploy/gbtw-watchdog.timer /etc/systemd/system/

# Edit WorkingDirectory and User in gbtw-watchdog.service if not using /opt/gbtw-promo-watchdog
sudo systemctl daemon-reload
sudo systemctl enable --now gbtw-watchdog.timer
```

Check timer status and logs anytime:
```bash
systemctl list-timers gbtw-watchdog.timer
journalctl -u gbtw-watchdog.service -n 50
```

---

## Disclaimer
This project is an independent community utility and is not affiliated with, endorsed by, or sponsored by JOYCITY Corporation.
