# Discord Bot Starter Kit - Buyer Guide

## 📍 Where to Start

**New to this product?** Follow this path:
1. **Read this guide** → Understand what you're buying
2. [Installation Guide](INSTALLATION.md) → Step-by-step setup
3. [Buyer Acceptance Checks](BUYER_ACCEPTANCE_CHECKS.md) → Verify everything works
4. [Production Deployment](PRODUCTION_DEPLOYMENT.md) → Go live
5. [Operations Guide](OPERATIONS.md) → Day-to-day management

**Having issues?** Jump to [Troubleshooting Guide](TROUBLESHOOTING.md)

---

## 🎯 What You're Buying

A **production-ready Discord bot framework** with everything wired together:

- ✅ **Discord Bot Core** - Slash commands, permissions, admin panel
- ✅ **Stripe Payments** - Checkout, subscriptions, webhook handling
- ✅ **Demo Mode** - Full premium simulation (no Stripe required)
- ✅ **Database** - SQLite with migrations and user tracking
- ✅ **Background Tasks** - Redis + Celery for async operations
- ✅ **Feature Flags** - Control features without redeployment
- ✅ **Docker Ready** - Complete containerization with nginx
- ✅ **Test Suite** - 30+ tests covering critical flows
- ✅ **Docs** - Full documentation suite

## 🚀 Quick Start (5 Minutes)

### Prerequisites
- Python 3.10+
- Discord account & server
- (Optional) Stripe account for payments

### 1. Create Discord Bot
1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. New Application → Add Bot → Reset Token (copy it)
3. Enable: Message Content Intent, Server Members Intent
4. OAuth2 → URL Generator → select `bot` + `applications.commands`
5. Add permissions: Send Messages, Use Slash Commands, Embed Links
6. Copy URL and invite bot to your server

### 2. Configure Environment
```bash
cp .env.example .env.dev
```

Edit `.env.dev`:
```env
APP_ENV=dev
DISCORD_BOT_TOKEN_DEV=your_bot_token_here
FEATURE_DEMO_MODE=true
```

### 3. Install & Run
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python -m src.app.discord_bot_app
```

### 4. Test Commands
```
/ping      → Bot status
/demo      → Interactive demo
/pricing   → Pricing flow (simulated)
/status    → System health
```

## 💰 Monetization Architecture

This product has **two independent billing layers**. They do not automatically synchronize.

### Layer 1 — Discord Bot User Premium

Controls bot command access (`/premium`, `/demo_feature`) for individual Discord users.

| Mode | How it works |
|---|---|
| **Demo** | User runs `/demo`; demo state persisted in `user_profile.demo_premium` (no Stripe) |
| **Real premium** | User pays via `/pricing` → Stripe → webhook sets `user_profile.is_premium` |

Requires: `FEATURE_PAYMENTS=true`, Stripe keys, a **publicly accessible webhook endpoint** (`/webhooks/stripe/webhook`) to receive payment confirmation.

### Layer 2 — SaaS Tenant Platform Billing

Controls the dashboard plan tier (`free`, `starter`, `pro`, `enterprise`) for the SaaS account holder.

- Managed via the web dashboard Billing page
- Stored in `saas_billing` table
- **Does not grant Discord bot premium to any Discord user**

### Demo Mode (Development & Testing)
- Set `FEATURE_DEMO_MODE=true` to enable
- No Stripe required; simulates the full premium flow
- Demo state is persisted in the database (survives bot restarts)
- Safe for demos and development

## 📋 What's Included

| Component | Status | Description |
|-----------|--------|-------------|
| **Bot Core** | ✅ Complete | Discord.py 2.x with slash commands |
| **Database** | ✅ Complete | SQLite with automatic migrations |
| **Payments** | ✅ Complete | Stripe integration with webhooks |
| **Admin** | ✅ Complete | Discord-native admin commands |
| **Docker** | ✅ Complete | Multi-service containerization |
| **Tests** | ✅ Complete | 30+ automated tests |
| **Docs** | ✅ Complete | Full documentation suite |

## 🔧 Customization Guide

### Rename Your Bot
```env
BOT_NAME=My Awesome Bot
PREMIUM_BRAND_NAME=My Bot Pro
```

### Add Commands
1. Create file in `src/app/discord_handlers/`
2. Register in `src/app/discord_bot_app.py`
3. Restart the bot; Discord slash commands require a sync on startup (not auto-reloaded at runtime)

### Database Changes
Edit `src/app/db/schema.py` - tables auto-create on start

## 🎨 Feature Flags

Control features without redeployment:
```env
FEATURE_PAYMENTS=true     # Enable/disable Stripe
FEATURE_ADMIN=true        # Enable admin commands
FEATURE_DEMO_MODE=false   # Enable demo mode
```

## 📦 Deployment Options

### Option 1: Docker (Recommended)
```bash
docker-compose up -d
```
Includes: Bot, API, Redis, Celery, nginx

### Option 2: VPS/Server
```bash
# Bot only
python -m src.app.discord_bot_app

# Or with systemd
sudo systemctl enable discord-bot
```

### Option 3: Cloud Platform
- AWS ECS/Azure Container Apps
- Google Cloud Run
- Heroku (with addons)

## 🔒 Security Features

- ✅ Environment variable validation
- ✅ Test/live key separation
- ✅ Webhook signature verification
- ✅ Admin access control
- ✅ Rate limiting ready
- ✅ Error handling & logging

## 📊 Analytics & Monitoring

Built-in tracking:
- User interactions
- Command usage
- Payment events
- System health
- Error logs

## 🌍 Multi-Language Support

Included translations:
- English (en)
- Slovak (sk)
- Czech (cs)
- German (de)
- French (fr)
- Spanish (es)

## 🧪 Testing

Run the test suite:
```bash
# All tests
pytest

# Critical tests only
pytest -m gate

# Skip slow tests
pytest -m "not slow"
```

## 📞 Support

- **Documentation**: `docs/` directory
- **Examples**: Check `.env.example`
- **Tests**: `tests/` directory
- **Health Check**: Run `/status` command

## 🎯 Next Steps

1. **Check the docs** — start with [Troubleshooting Guide](TROUBLESHOOTING.md)
2. **Customize the bot** with your branding
3. **Set up Stripe** for real payments
4. **Deploy to production** using Docker
5. **Monitor analytics** via admin commands

---

**Created and maintained by Janko Duris - Dulvarn**  
Commercial License - See LICENSE and TERMS.md for details
