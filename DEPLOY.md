# Render Deployment Guide — عقاري الذكي

This project deploys as **two separate Render services** from the same repository.

---

## Prerequisites

- A Render account (render.com)
- An external PostgreSQL database (Render Postgres, Supabase, Neon, etc.)
- All secrets listed below

---

## Services

| Service | Type | Start command |
|---|---|---|
| `aqari-bot` | Background Worker | `python telegram-bot/bot.py` |
| `aqari-dashboard` | Web Service | `gunicorn --chdir telegram-bot dashboard:app ...` |

Both services share the same repository and the same `requirements.txt`.

---

## Step-by-step deployment

### 1. Connect your repository

In the Render dashboard → **New** → **Blueprint** → connect your GitHub/GitLab repo.  
Render will detect `render.yaml` and create both services automatically.

**OR** create them manually:

**Bot worker:**
- Type: Background Worker
- Build command: `pip install -r telegram-bot/requirements.txt`
- Start command: `python telegram-bot/bot.py`

**Dashboard web service:**
- Type: Web Service
- Build command: `pip install -r telegram-bot/requirements.txt`
- Start command: `gunicorn --chdir telegram-bot dashboard:app --bind 0.0.0.0:$PORT --workers 2 --timeout 60`

---

### 2. Set environment variables

Set these on **both** services unless noted:

| Variable | Required on | Description |
|---|---|---|
| `DATABASE_URL` | Both | PostgreSQL connection string (e.g. from Render Postgres) |
| `TELEGRAM_BOT_TOKEN` | Bot only | From @BotFather |
| `OPENAI_API_KEY` | Bot only | OpenAI API key |
| `ADMIN_CHAT_ID` | Bot only | Your Telegram chat ID (use /myid in the bot) |
| `PUBLISH_CHANNEL_USERNAME` | Bot only | e.g. `@AgariSmartOffers` |
| `ADMIN_PASSWORD` | Dashboard only | Login password for the web dashboard |
| `SESSION_SECRET` | Dashboard only | Any long random string |
| `GOOGLE_SHEETS_CREDENTIALS_JSON` | Bot only | Full JSON content of the service account key |
| `GOOGLE_SHEET_ID` | Bot only | Spreadsheet ID from the Google Sheets URL |

> **Tip:** For `DATABASE_URL`, use the **Internal** connection string if both services are on the same Render region, otherwise use External.

---

### 3. Database

The bot automatically runs `init_db()` on startup, which creates all required tables if they do not exist. No manual migration is needed.

Supported: any PostgreSQL database reachable via `DATABASE_URL`.

---

### 4. Python version

Render uses Python **3.11** by default (matching this project).  
To pin it explicitly, add a `runtime.txt` file:

```
python-3.11.14
```

---

### 5. Verify deployment

After both services are live:

1. Send `/start` to your Telegram bot — it should respond
2. Open the dashboard URL Render provides → log in with `ADMIN_PASSWORD`
3. Submit a test listing or request via the bot → check the dashboard and Google Sheets

---

## Local development

```bash
cd telegram-bot

# Install dependencies
pip install -r requirements.txt

# Run the bot
python bot.py

# Run the dashboard
python dashboard.py
```

Both services read all configuration from environment variables. You can use a `.env` file with `python-dotenv` locally, or export variables in your shell.
