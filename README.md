<div align="center">

# 🔐 IDentifyou

**Discord verification bot with image CAPTCHA, persistent storage and bilingual support.**

[![discord.js](https://img.shields.io/badge/discord.js-v14-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.js.org)
[![Node.js](https://img.shields.io/badge/Node.js-≥18-339933?style=flat-square&logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)
[![Language](https://img.shields.io/badge/language-EN%20%2F%20DE-blue?style=flat-square)](#-bilingual-support)

[**Add to Discord →**](https://discord.com/oauth2/authorize?client_id=1477409638709461073)

</div>

---

## ✨ Features

| | Feature | Description |
|---|---|---|
| 🖼️ | **Image CAPTCHA** | Distorted characters, noise & color randomization — generated server-side |
| 📬 | **DM-first flow** | CAPTCHA sent via DM, falls back to ephemeral if DMs are closed |
| 🔒 | **Auto channel locking** | Sets `@everyone ViewChannel: false` on all channels without role overrides |
| 🎭 | **Auto role creation** | Creates `Verified` & `Unverified` roles automatically if none are provided |
| 💾 | **Persistent storage** | Everything saved in `db.json` with atomic writes — survives restarts & crashes |
| 🌐 | **Bilingual** | Full English & German support, switchable per server with `/language` |
| 📊 | **Statistics** | Track verifications, failed attempts, success rate & active sessions |
| ⚙️ | **Zero-config setup** | Run `/setup` with no options — everything is created automatically |
| 🎨 | **Fully customizable** | Embed title, description, color, button label & style — all configurable |
| 🎯 | **Attempt limiting** | Configurable max attempts before timeout, auto-expiring sessions |

---

## 🚀 Self-Hosting

### Prerequisites

```bash
# Node.js ≥ 18
node --version

# Canvas native dependencies (Ubuntu / Debian)
sudo apt install -y build-essential libcairo2-dev \
  libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev
```

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/AufGekaut/IDentifyou.git
cd IDentifyou

# 2. Install dependencies
npm install

# 3. Configure environment
nano .env
```

### Configuration

Edit `.env` — only `TOKEN` and `CLIENT_ID` are required:

```env
TOKEN=your_bot_token_here
CLIENT_ID=your_application_id_here
```

All other values are optional and have sensible defaults. See the [full `.env` reference](#-env-reference) below.

### Start

```bash
# Development
node src/index.js

# Production (PM2 recommended)
npm install -g pm2
pm2 start src/index.js --name identifyou
pm2 save
pm2 startup
```

---

## 🛠️ Commands

All commands require the **Administrator** permission.

| Command | Description |
|---|---|
| `/setup` | Configure verification — all options optional, everything auto-created if left empty |
| `/language` | Switch bot language: `en` (English, default) or `de` (Deutsch) |
| `/status` | Show current configuration (channels, roles, colors, language) |
| `/stats` | View verification statistics (total, failed, success rate, active sessions) |
| `/reset` | Clear stored config — channels and roles are **not** deleted |

### `/setup` Options

| Option | Type | Description |
|---|---|---|
| `channel` | Channel | Existing channel for the verify panel (empty = auto-create `#verify`) |
| `verified_role` | Role | Role granted after verification (empty = auto-create `Verified`) |
| `unverified_role` | Role | Role assigned to new members (empty = auto-create `Unverified`) |
| `log_channel` | Channel | Channel for verification logs (empty = auto-create `#verify-logs`) |
| `embed_title` | String | Title of the verification embed |
| `embed_description` | String | Description of the verification embed |
| `embed_color` | String | Hex color e.g. `#5865F2` |
| `button_label` | String | Label on the verify button |
| `button_color` | Choice | `Primary` · `Secondary` · `Success` · `Danger` |

> **Tip:** Running `/setup` with no options creates everything automatically and locks all channels.

---

## ⚙️ `.env` Reference

```env
# ── Required ──────────────────────────────────
TOKEN=                        # Bot token from Discord Developer Portal
CLIENT_ID=                    # Application ID

# ── CAPTCHA ───────────────────────────────────
CAPTCHA_LENGTH=6              # Characters in CAPTCHA (4–8 recommended)
CAPTCHA_EXPIRE_MINUTES=5      # Minutes before CAPTCHA expires
CAPTCHA_MAX_ATTEMPTS=3        # Max wrong attempts (0 = unlimited)

# ── Embed ─────────────────────────────────────
EMBED_COLOR=#5865F2
EMBED_TITLE=🔐 Server Verification
EMBED_DESCRIPTION=Click the button below to verify yourself.
EMBED_FOOTER=Verification powered by IDentifyou

# ── Button ────────────────────────────────────
BUTTON_LABEL=✅ Verify me
BUTTON_COLOR=Success          # Primary | Secondary | Success | Danger

# ── Messages ──────────────────────────────────
MSG_SUCCESS=✅ **Successfully verified!** Welcome to the server!
MSG_FAIL=❌ Wrong CAPTCHA. Please try again.
MSG_TIMEOUT=⏰ CAPTCHA expired. Please click the button again.
MSG_MAX_ATTEMPTS=🚫 Too many wrong attempts. Please try again later.
```

---

## 🌐 Bilingual Support

IDentifyou supports **English** (default) and **German** per server. Every embed, button, modal and message is fully translated.

```bash
/language de   # Switch to German
/language en   # Switch back to English
```

Language is stored per-guild in `db.json` and persists across restarts.

---

## 💾 Persistent Storage

All data is stored in a single `db.json` file using **atomic writes** (write to `.tmp` → rename) to prevent corruption on crash.

```json
{
  "guilds":   { "<guildId>": { "verifyChannelId": "...", "verifiedRoleId": "...", ... } },
  "sessions": { "<userId>":  { "code": "AB3X7Z", "attempts": 0, "expiresAt": 123456789 } },
  "stats":    { "<guildId>": { "total": 42, "failed": 7, "lastVerified": { ... } } }
}
```

---

## 🔒 How Channel Locking Works

When `/setup` runs, IDentifyou scans **all text and voice channels**:

- Does the channel have **no custom role overrides**?
- Can `@everyone` currently see it?

If both are true → `@everyone ViewChannel: false` + `Verified ViewChannel: true` is applied.

Channels with existing role configurations are **never touched**.

---

## 📁 File Structure

```
IDentifyou/
├── src/
│   ├── index.js                  # Entry point, event loader
│   ├── captcha.js                # Image CAPTCHA generator
│   ├── db.js                     # Persistent storage (db.json)
│   ├── i18n.js                   # EN / DE translations
│   ├── commands/
│   │   ├── setup.js              # /setup
│   │   ├── language.js           # /language
│   │   ├── status.js             # /status
│   │   ├── stats.js              # /stats
│   │   └── reset.js              # /reset
│   └── events/
│       ├── ready.js              # Register slash commands on startup
│       ├── interactionCreate.js  # Button / modal handler (core verify flow)
│       └── guildMemberAdd.js     # Assign Unverified role to new members
├── db.json                       # Auto-created on first run
├── .env                          # Configuration (never commit this!)
├── package.json
└── README.md
```

---

## 🤖 Required Bot Permissions

| Permission | Why |
|---|---|
| `Manage Roles` | Assign Verified / Unverified roles |
| `Manage Channels` | Create and lock channels |
| `Send Messages` | Post the verify panel and logs |
| `Embed Links` | Send rich embeds |
| `Attach Files` | Send CAPTCHA images |
| `Read Message History` | Clear old panel messages on re-setup |
| `View Channels` | Access channels to configure them |

> ⚠️ The bot's role must be **above** the `Verified` and `Unverified` roles in the server's role hierarchy.

---

## 📦 Deploy with PM2

```bash
# Install PM2
npm install -g pm2

# Start bot
pm2 start src/index.js --name identifyou

# Auto-restart on server reboot
pm2 save
pm2 startup

# Useful commands
pm2 status                  # Check if running
pm2 logs identifyou         # View live logs
pm2 restart identifyou      # Restart after code changes
```

### Updating

```bash
git pull
pm2 restart identifyou
```

---

## 📄 License

MIT — do whatever you want, just don't blame me if something breaks.
