# عقاري الذكي — Arabic Real Estate Broker Platform

## Overview

A production-ready Arabic real estate broker system with a Telegram bot for user input, AI-powered field extraction and ad generation, smart property matching, channel publishing, and a web admin dashboard.

## Stack

### Python (Main Application)
- **Runtime**: Python 3
- **Telegram Bot**: `python-telegram-bot`
- **AI**: OpenAI `gpt-4o-mini` (field extraction + ad generation)
- **Database**: PostgreSQL via `psycopg2-binary`
- **Admin Dashboard**: Flask + Bootstrap 5 (Arabic RTL)

### Node.js (Proxy Layer)
- **Monorepo**: pnpm workspaces
- **API Server**: Express 5 (proxies /admin → Flask on port 3000)

## Project Structure

```
telegram-bot/
  bot.py              — Main Telegram bot (guided forms, matching, notifications)
  dashboard.py        — Flask admin dashboard
  config.py           — Central config from environment variables
  models/
    database.py       — PostgreSQL connection + all DB operations
  services/
    ai_service.py     — OpenAI field extraction + ad generation
    matching.py       — Score-based property matching engine
    publishing.py     — Telegram channel publishing
  templates/          — Flask HTML templates (Arabic RTL)
    base.html, login.html, index.html,
    requests.html, listings.html, matches.html

artifacts/
  api-server/         — Express server that proxies /admin/* to Flask
```

## Environment Variables (Secrets)

| Variable | Required | Description |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | ✅ | Main bot token |
| `OPENAI_API_KEY` | ✅ | OpenAI API key |
| `ADMIN_CHAT_ID` | ✅ | Admin Telegram chat ID |
| `ADMIN_PASSWORD` | ✅ | Dashboard login password |
| `DATABASE_URL` | ✅ | PostgreSQL connection (auto-set by Replit) |
| `SESSION_SECRET` | ✅ | Flask session secret |
| `PUBLISH_CHANNEL_USERNAME` | ⬜ | e.g. `@AqariSmartOffers` for channel publishing |
| `PUBLISH_BOT_TOKEN` | ⬜ | Optional second bot token for publishing |

## Workflows

| Workflow | Command | Description |
|---|---|---|
| **Telegram Bot** | `python3 telegram-bot/bot.py` | Main bot (long-polling) |
| **Admin Dashboard** | `python3 telegram-bot/dashboard.py` | Flask on port 3000 (internal) |
| **artifacts/api-server** | `pnpm --filter @workspace/api-server run dev` | Express proxy, port 8080 (external) |

## Database Schema (PostgreSQL)

### `requests`
id, user_id, full_name, phone, city, district, sale_or_rent, property_type, rooms, budget_min, budget_max, notes, raw_text, ai_json, status, created_at

### `listings`
id, user_id, full_name, phone, role, city, district, sale_or_rent, property_type, area, rooms, price, description_raw, description_ai, images, ai_json, status, created_at

### `matches`
id, request_id, listing_id, match_score, match_reason, status, admin_notes, commission_value, notified, created_at

## Admin Dashboard Access

The Flask dashboard (port 3000) is accessed through the Express API server proxy:
- **Preview pane**: select "Admin Dashboard" workflow
- **URL**: `https://<dev-domain>/admin/login`

## Bot Commands

### User Commands
- `/start` — Main menu with "عندي طلب" / "عندي عرض" buttons

### Admin-Only Commands (ADMIN_CHAT_ID only)
- `/admin_requests` — Latest 10 requests
- `/admin_listings` — Latest 10 listings
- `/admin_matches` — Latest 10 matches
- `/admin_stats` — Total counts
- `/admin_delete_old` — Delete records older than 30 days

## Matching Engine

Scores are computed per match (max 100):
- City match: 40 pts
- District match: 20 pts
- Property type match: 20 pts
- Sale/rent match: 10 pts
- Rooms within ±1: 10 pts

Threshold: ≥ 60 → save match + notify

## Key Design Decisions

- Broker remains middleman — phone numbers never shared between users
- Users press "طلب تواصل من الوسيط" → admin receives full lead details
- AI fallback: if OpenAI fails, template-based ad is used
- Channel publishing fallback: if PUBLISH_CHANNEL_USERNAME not set, skip silently
- All DB operations are migration-safe (CREATE TABLE IF NOT EXISTS)
