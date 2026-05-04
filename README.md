# Discord Bot Starter Kit

A production-grade Discord bot framework with Stripe subscriptions, multi-tenant SaaS infrastructure, and complete deployment automation.

**What this is:** A foundational framework for building monetized Discord bots with subscription management, multi-tenant support, and a web dashboard for guild configuration.

**What this is not:** A turnkey bot you can deploy without customization. This is a framework requiring Python/Discord.py knowledge to extend with your bot's business logic.

Created and maintained by [Janko Duris - Dulvarn](https://www.dulvarn.com).

---

## 💰 Get This Kit

**$129** — One-time payment. Lifetime access. All future updates included.

👉 [**Buy on Gumroad**](https://jankodur.gumroad.com/l/iwsfui)

> After purchase you receive a license key → GitHub access to this private repo via
> Outside Collaborator invite (automated within minutes).

---

## What's Included

| Area | What You Get |
|------|-------------|
| **Discord Bot** | discord.py slash commands, tenant-aware handlers, premium gating example (`/server_report`) |
| **FastAPI Backend** | REST API, Stripe webhooks, Discord OAuth, SaaS dashboard, email magic links |
| **Stripe Billing** | Subscription checkout, Customer Portal, webhook handler, plan enforcement |
| **Multi-tenant SaaS** | Tenant isolation, invitations, per-tenant config, role-based access |
| **Database** | SQLite (dev) + PostgreSQL (prod) with Alembic migrations |
| **Background Tasks** | Celery + Redis — email, analytics, backups, scheduled jobs |
| **Auth** | Email/password + magic link + Discord OAuth |
| **Dashboard** | Vanilla JS SaaS web UI — billing, Discord connection, settings |
| **Tests** | 40+ pytest tests across all modules |
| **Docs** | Architecture, deployment, Stripe setup, buyer guides |
| **Docker** | docker-compose for dev + prod, nginx config, systemd service |

---

## Who This Is For

✅ Python developers building monetized Discord bots
✅ Indie hackers launching a bot-based SaaS
✅ Teams needing multi-tenant Discord infrastructure fast

❌ Not for: no-code builders, beginners without Python knowledge,
   teams wanting a ready-to-deploy bot without customization

---

> ⚠️ **FRAMEWORK NOTICE**
> This is an **infrastructure framework**, not a turnkey bot. It includes complete
> Stripe billing, multi-tenancy, SaaS dashboard, and Discord integration.
> **You must implement your bot's actual business logic.** Premium commands include
> one working example (`/server_report`) to demonstrate the pattern.

---

## Demo

> Dashboard preview and bot commands demo:
> 📺 [Watch 5-minute demo](https://www.youtube.com/watch?v=PLACEHOLDER)
> *(Replace PLACEHOLDER with your YouTube/Loom URL after recording)*

---

## Architecture Overview

This repository provides a multi-service architecture:

```
┌─────────────────┐
│   Discord Bot   │  ← discord.py slash commands, tenant-aware command handlers
│   (discord_bot_app.py)
└────────┬────────┘
         │
         ├─────────────────────────────────────────────┐
         │                                             │
┌────────▼────────┐                          ┌────────▼────────┐
│  FastAPI Server │  ← Webhooks, SaaS API, OAuth     │  SQLite/Postgres │  ← Users, subscriptions,
│  (api/main.py)  │    tenant management, dashboard   │    Database      │    events, tenant config
└────────┬────────┘                          └─────────────────┘
         │
         ├─────────────────────────────────────────────┐
         │                                             │
┌────────▼────────┐                          ┌────────▼────────┐
│  Redis + Celery │  ← Background tasks, caching      │     nginx       │  ← SSL termination,
│                 │    scheduled jobs, rate limiting   │   Reverse Proxy │    load balancing
└─────────────────┘                          └─────────────────┘
```

### Service Boundaries

| Service | Responsibility | Required For |
|---------|---------------|--------------|
| **Discord Bot** | Discord API interaction, slash commands, tenant context resolution | All deployments |
| **FastAPI API** | Stripe webhooks, SaaS REST API, OAuth callbacks, static dashboard | Payments + SaaS dashboard |
| **Redis** | Celery broker, rate limiting, caching | Background tasks + rate limiting |
| **Celery Worker** | Async task processing (email, cleanup, analytics) | Background tasks |
| **Celery Beat** | Scheduled tasks (backups, maintenance) | Scheduled jobs |
| **nginx** | SSL termination, reverse proxy, static file serving | Production deployments |
| **PostgreSQL** | Production database (alternative to SQLite) | High-concurrency deployments |

### Deployment Options

1. **Bot-only deployment** (no payments, no dashboard): Discord Bot + SQLite
2. **Payments deployment** (Stripe webhooks): Discord Bot + FastAPI API + SQLite + nginx
3. **Full SaaS deployment** (multi-tenant): All services including PostgreSQL, Redis, Celery

See [Architecture Documentation](docs/ARCHITECTURE.md) for detailed component boundaries and data flows.

---

## Quick Start (Bot-Only Deployment)

### Prerequisites

- Python 3.10+
- Discord Developer account
- Discord server for testing

### 1. Discord Bot Setup

Create a Discord application and bot:

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create application → navigate to **Bot** → **Add Bot**
3. Enable **Message Content Intent** and **Server Members Intent**
4. Copy the bot token

For detailed Discord app setup including OAuth scopes, see [Discord App Setup Guide](docs/DISCORD_APP_SETUP.md).

### 2. Environment Configuration

```bash
cp .env.example .env.dev
```

Edit `.env.dev` with minimum configuration:

```env
APP_ENV=dev
DISCORD_BOT_TOKEN_DEV=your_bot_token_here
FEATURE_DEMO_MODE=true
```

### 3. Install and Run

```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python -m src.app.discord_bot_app
```

### 4. Test Commands

In your Discord server:
- `/ping` - Bot health check
- `/demo` - Interactive demo (requires demo mode enabled)
- `/status` - System status

---

## Documentation

| Document | Purpose |
|----------|---------|
| [Technical Buyer Guide](docs/TECHNICAL_BUYER_GUIDE.md) | Quick evaluation guide for technical decision-makers |
| [Architecture](docs/ARCHITECTURE.md) | Component boundaries, data flows, service interactions |
| [Environment Variables](docs/ENVIRONMENT_VARIABLES.md) | Complete reference for all configuration |
| [Discord App Setup](docs/DISCORD_APP_SETUP.md) | Discord Developer Portal configuration |
| [Stripe Setup](docs/STRIPE_SETUP.md) | Stripe products, prices, webhooks |
| [Extension Points](docs/EXTENSION_POINTS.md) | Customization and extension guide |
| [Production Readiness](docs/PRODUCTION_READINESS.md) | Pre-deployment checklist |
| [Installation](docs/INSTALLATION.md) | Step-by-step installation |
| [Production Deployment](docs/PRODUCTION_DEPLOYMENT.md) | Deployment strategies and procedures |
| [Operations](docs/OPERATIONS.md) | Day-to-day operations and maintenance |
| [Security](docs/SECURITY.md) | Security best practices and checklist |
| [Billing Troubleshooting](docs/BILLING_TROUBLESHOOTING.md) | Stripe payment issues and resolution |

## Limitations

This framework has the following technical limitations:

- **Single bot instance per deployment** - Discord does not allow horizontal scaling of a single bot token. Each deployment runs one bot instance connected to one bot token.
- **SQLite write locks** under high concurrency - For production deployments with concurrent users, migrate to PostgreSQL.
- **Discord API rate limits** - The bot respects Discord's rate limits automatically. For high-volume bots, implement command queuing.
- **No built-in sharding** - For deployments to 1000+ servers, implement Discord bot sharding architecture (not included).
- **Webhook endpoint accessibility** - Stripe webhooks require a publicly accessible HTTPS endpoint. Local development requires Stripe CLI or tunneling.
- **Two independent billing layers** - Discord user premium and SaaS tenant billing are separate systems with no automatic synchronization.

For detailed technical evaluation, see [Technical Buyer Guide](docs/TECHNICAL_BUYER_GUIDE.md).

---

## Billing Architecture

This product contains **two independent billing layers** that do not synchronize automatically:

| Layer | Database Table | Controls | Paid Via |
|-------|---------------|----------|----------|
| **Discord User Premium** | `user_profile` (`is_premium`, `demo_premium`) | Bot command access (`/premium`, `/demo_feature`) | Discord users via `/pricing` → Stripe |
| **SaaS Tenant Billing** | `saas_billing` (`plan_tier`) | Dashboard plan tier (`free`, `starter`, `pro`, `enterprise`) | Dashboard users via Billing page → Stripe |

**Important:** Upgrading the SaaS platform plan does not grant Discord bot premium to any Discord user. The two flows use separate Stripe endpoints and database tables.

### Discord User Premium Flow

1. User runs `/pricing` → selects plan
2. Bot creates Stripe checkout session with `discord_user_id` in metadata
3. User completes payment on Stripe checkout
4. Stripe webhook hits `/webhooks/stripe/webhook`
5. Webhook verifies signature → sets `user_profile.is_premium = 1`
6. User gains premium access on guilds where `feature_premium` is enabled

**Requirement:** Webhook endpoint must be publicly reachable (HTTPS). For local testing, use Stripe CLI to forward webhooks.

### Tenant-Aware Configuration

Each Discord guild can be mapped to a SaaS tenant. When a command is invoked:
1. Bot resolves guild ID from interaction
2. Looks up guild in `saas_discord_connections` → finds linked tenant
3. Reads `tenant_config` for per-tenant feature flags and display settings
4. Tenant-level flags override global `.env` flags

If no tenant mapping exists, bot falls back to global `.env` feature flags.

Per-tenant settings (managed via web dashboard):
- `feature_demo_mode` - Enable/disable `/demo` and `/demo_feature`
- `feature_premium` - Enable/disable `/premium` command gate
- `feature_admin` - Enable/disable `/admin_stats`
- `bot_display_name` - Shown in bot embed footers
- `welcome_message` - Prepended to `/demo` description

---

## Technical Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Python** | 3.10 | 3.11+ |
| **RAM** | 512MB | 2GB (full stack) |
| **Storage** | 100MB | 10GB SSD |
| **Database** | SQLite (built-in) | PostgreSQL for production |
| **Redis** | Optional (for rate limiting) | Required for Celery + rate limiting |

### Dependencies

Core dependencies:
- `discord.py` >= 2.3.0 - Discord API wrapper
- `fastapi` >= 0.104.0 - API server
- `stripe` >= 7.0.0 - Stripe SDK
- `celery` >= 5.3.0 - Background tasks
- `redis` >= 5.0.0 - Caching and queue broker

Full dependency list in `requirements.txt`.

---

## Demo Mode

Demo mode enables the full premium flow without Stripe configuration:

```env
FEATURE_DEMO_MODE=true
```

In demo mode:
- `/demo` runs interactive walkthrough
- `/pricing` shows simulated checkout
- Premium commands respond as if user has subscription
- No Stripe API calls, no real charges

Demo state persists in database (`user_profile.demo_premium`), surviving bot restarts.

---

## Feature Flags

Control features without redeployment via environment variables:

```env
FEATURE_PAYMENTS=true     # Stripe integration
FEATURE_ADMIN=true        # Admin commands
FEATURE_DEMO_MODE=false   # Demo mode
```

The `FeatureFlagManager` supports percentage rollouts, whitelists, gradual rollouts, and A/B testing. See `src/app/feature_flags/flags.py` for implementation.

---

## License

Commercial license. See [LICENSE](./LICENSE) and [TERMS.md](./TERMS.md).

**Summary:**
- Use this software to build and sell your own Discord bot products
- Do NOT redistribute or resell the source code
- Do NOT use this to create a competing template

| License | Price | Use |
|---------|-------|-----|
| Solo | $49 | 1 project, 1 developer |
| Studio | $149 | 5 projects, 3 developers |
| White-Label | $499 | Unlimited projects, resell allowed |

Purchase at: https://www.dulvarn.com