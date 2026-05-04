# Architecture Documentation

## Overview

The Discord Bot Starter Kit is a multi-service architecture supporting three deployment modes:

1. **Bot-only**: Discord Bot + SQLite
2. **Payments**: Discord Bot + FastAPI + SQLite + nginx
3. **Full SaaS**: All services including PostgreSQL, Redis, Celery

## Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        External Services                        │
├──────────────────────┬──────────────────────┬───────────────────┤
│   Discord Gateway    │      Stripe API      │   Web Browser     │
│   (WebSocket)        │      (HTTPS)         │   (HTTPS)         │
└──────────┬───────────┴──────────┬───────────┴─────────┬─────────┘
           │                      │                     │
           │                      │                     │
┌──────────▼──────────────────────▼─────────────────────▼─────────┐
│                         nginx (Reverse Proxy)                   │
│                    SSL Termination / Load Balancing              │
└──────────┬──────────────────────┬─────────────────────┬─────────┘
           │                      │                     │
           │                      │                     │
┌──────────▼──────────┐  ┌────────▼────────┐  ┌────────▼─────────┐
│   Discord Bot       │  │  FastAPI Server │  │  Static Assets   │
│   (discord_bot_app) │  │  (api/main.py)  │  │  (static/)       │
└──────────┬──────────┘  └────────┬────────┘  └──────────────────┘
           │                      │
           │                      │
           │                      ├─────────────────────────────┐
           │                      │                             │
┌──────────▼──────────┐  ┌────────▼────────┐  ┌────────▼─────────┐
│   SQLite/Postgres   │  │     Redis       │  │  Celery Worker   │
│   (Database)        │  │   (Cache/Queue) │  │  (Background)    │
└─────────────────────┘  └─────────────────┘  └──────────────────┘
                                                        │
                                                        │
                                              ┌─────────▼─────────┐
                                              │  Celery Beat      │
                                              │  (Scheduler)      │
                                              └───────────────────┘
```

## Service Boundaries

### Discord Bot Service (`discord_bot_app.py`)

**Responsibilities:**
- Discord API interaction via discord.py
- Slash command registration and handling
- Tenant context resolution for each guild
- User tracking and event logging
- Premium access enforcement

**Database Access:**
- Reads: `users`, `user_profile`, `tenant_config`, `saas_discord_connections`
- Writes: `users`, `user_profile`, `events`

**External Dependencies:**
- Discord Gateway (WebSocket)
- Database (SQLite/PostgreSQL)
- (Optional) Redis for rate limiting

**Does NOT access:**
- Stripe API (webhooks handled by API service)
- SaaS authentication (handled by API service)

**Entry Points:**
- `src/app/discord_bot_app.py` - Main bot entry point
- `src/app/discord_handlers/` - Command handlers

### FastAPI Service (`api/main.py`)

**Responsibilities:**
- Stripe webhook processing (`/webhooks/stripe/webhook`)
- SaaS REST API for tenant management
- OAuth callback handling (Discord OAuth)
- Static web dashboard serving
- Health check endpoints

**Database Access:**
- Full access to all tables
- Writes to `stripe_events` for idempotency
- Updates `user_profile.is_premium` on payment
- Manages `saas_tenants`, `saas_billing`, `saas_discord_connections`

**External Dependencies:**
- Stripe API (webhook verification, checkout creation)
- Database (SQLite/PostgreSQL)
- (Optional) Redis for caching
- (Optional) SMTP for email

**Routers:**
- `webhooks` - Stripe webhook endpoints
- `auth` - Authentication (login, register, password reset)
- `tenant_router` - Tenant management
- `billing` - SaaS billing endpoints
- `discord_oauth_router` - Discord OAuth integration
- `settings` - Tenant configuration
- `runtime` - Runtime tenant context resolution
- `onboarding` - Tenant onboarding
- `internal` - Internal diagnostics (protected by INTERNAL_ADMIN_TOKEN)

### Redis Service

**Responsibilities:**
- Celery message broker
- Rate limiting backend
- Application caching
- Session storage (optional)

**Data Structures:**
- Celery task queues: `default`, `broadcast`, `notifications`, `maintenance`, `analytics`, `backup`
- Rate limit keys: `rate_limit:{user_id}:{command}`
- Cache keys: `user_profile:{user_id}`, `tenant_config:{tenant_id}`

**Persistence:**
- AOF (Append Only File) enabled by default
- Snapshot every second (configurable)

### Celery Worker Service

**Responsibilities:**
- Async task processing
- Email sending (password reset, verification)
- Database cleanup and maintenance
- Analytics aggregation
- Backup operations

**Task Queues:**
- `default` - General async tasks
- `broadcast` - Bulk notifications
- `notifications` - Individual notifications
- `maintenance` - Database cleanup, optimization
- `analytics` - Analytics aggregation
- `backup` - Database backups

**Entry Point:**
- `celery_worker.py` - Celery application definition

### Celery Beat Service

**Responsibilities:**
- Scheduled task execution
- Periodic backups
- Maintenance jobs
- Analytics rollups

**Scheduled Tasks:**
- Daily database backup (configurable)
- Weekly database vacuum
- Monthly analytics aggregation

### nginx Service

**Responsibilities:**
- SSL/TLS termination
- Reverse proxy to FastAPI
- Static file serving
- Rate limiting (HTTP level)
- Security headers

**Configuration:**
- `nginx/nginx.conf` - Main configuration
- `nginx/ssl/` - SSL certificates

## Data Flow

### Discord User Premium Payment Flow

```
User runs /pricing
    ↓
Discord Bot creates Stripe checkout session
    ↓
Bot returns checkout URL to user
    ↓
User completes payment on Stripe
    ↓
Stripe sends webhook to /webhooks/stripe/webhook
    ↓
FastAPI verifies webhook signature
    ↓
FastAPI checks stripe_events table for idempotency
    ↓
FastAPI updates user_profile.is_premium = 1
    ↓
FastAPI returns 200 OK to Stripe
    ↓
User gains premium access on next command
```

### Tenant-Aware Command Execution Flow

```
User runs /command in Discord
    ↓
Discord Bot receives interaction
    ↓
Bot extracts guild_id from interaction
    ↓
Bot queries saas_discord_connections for guild_id
    ↓
If mapped: Bot loads tenant_config from saas_tenants
If not mapped: Bot uses global .env feature flags
    ↓
Bot checks feature_premium flag for guild
    ↓
Bot checks user_profile.is_premium for user
    ↓
Bot executes command with resolved context
```

### SaaS Tenant Billing Flow

```
Admin accesses dashboard
    ↓
Dashboard calls /api/auth/login
    ↓
FastAPI validates credentials, returns JWT
    ↓
Dashboard calls /api/billing/subscribe
    ↓
FastAPI creates Stripe checkout session
    ↓
Admin completes payment on Stripe
    ↓
Stripe sends webhook to /webhook (SaaS billing endpoint)
    ↓
FastAPI updates saas_billing.plan_tier
    ↓
Tenant features unlocked based on plan
```

## Database Schema

### Core Tables

**users**
- Discord user records
- Basic profile information

**user_profile**
- User premium status (is_premium, demo_premium)
- Stripe subscription IDs
- Language preference
- Guest/registered status

**events**
- Event logging for analytics
- User interactions
- System events

**stripe_events**
- Idempotency table for Stripe webhooks
- Prevents duplicate processing
- Status tracking (processing, completed, failed)

### SaaS Tables

**saas_tenants**
- Tenant records
- Tenant configuration JSON
- Feature flags per tenant

**saas_billing**
- Tenant billing status
- Plan tier (free, starter, pro, enterprise)
- Stripe subscription IDs

**saas_discord_connections**
- Maps Discord guilds to tenants
- Enables per-guild configuration

**saas_users**
- SaaS dashboard users
- Email/password authentication
- JWT tokens

### Content Tables

**content_items**
- Premium/free content
- Tier-based access control

**content_downloads**
- Download tracking
- Delivery method logging

## Extension Points

### Adding Discord Commands

Location: `src/app/discord_handlers/`

1. Create new handler file
2. Define command function with `@tree.command` decorator
3. Register in `discord_bot_app.py` setup_hook
4. Commands automatically inherit tenant-aware context

### Adding API Endpoints

Location: `src/app/api/`

1. Create router module
2. Define FastAPI routes
3. Include router in `api/main.py`
4. Add authentication middleware if needed

### Adding Celery Tasks

Location: `src/app/tasks/` or inline in handlers

```python
from src.app.tasks.celery_app import celery_app

@celery_app.task(queue='default')
def my_async_task(user_id: int):
    # Task implementation
    pass
```

### Adding Database Tables

Location: `src/app/db/schema.py`

1. Add CREATE TABLE statement to SCHEMA_SQL
2. Database auto-migrates on next restart
3. Add query helpers in `database.py` if needed

## Security Boundaries

### Authentication Layers

1. **Discord Bot**: Discord user ID (implicit)
2. **SaaS API**: JWT tokens (email/password)
3. **Admin Commands**: ADMIN_USER_IDS whitelist
4. **Internal Endpoints**: INTERNAL_ADMIN_TOKEN header

### Data Isolation

- **Tenant data**: Isolated by tenant_id in saas_tenants
- **User data**: Isolated by user_id in user_profile
- **Guild data**: Isolated by guild_id in saas_discord_connections

### Secret Management

- All secrets in environment variables
- Never committed to git
- Production secrets in `/etc/discord-bot/prod.env` (chmod 600)
- Docker secrets support available

## Deployment Architectures

### Bot-Only (Development)

```
Discord Bot → SQLite
```

- No external dependencies
- No webhooks
- Suitable for local development and testing

### Payments (Small Production)

```
Discord Bot → SQLite
      ↓
FastAPI → Stripe Webhooks
      ↓
nginx (SSL)
```

- Supports real payments
- Webhook endpoint required
- Suitable for single-tenant deployments

### Full SaaS (Multi-Tenant Production)

```
Discord Bot → PostgreSQL
      ↓
FastAPI → Stripe Webhooks
      ↓
Redis → Celery Worker/Beat
      ↓
nginx (SSL)
```

- Supports multi-tenancy
- Background task processing
- High concurrency
- Suitable for commercial SaaS

## Scaling Considerations

### Vertical Scaling

- Increase RAM for larger databases
- Add CPU for more concurrent users
- PostgreSQL for better write performance

### Horizontal Scaling

- Discord Bot: Not horizontally scalable (single Discord connection per bot token)
- FastAPI: Can scale behind load balancer
- Celery: Can run multiple workers
- Redis: Can use Redis Cluster

### Bottlenecks

1. **SQLite**: Write locks under high concurrency → migrate to PostgreSQL
2. **Discord API**: Rate limits → implement command queuing
3. **Single Bot Instance**: Cannot scale horizontally → use sharding for large deployments

## Monitoring Points

### Health Checks

- `/health` - API service health
- `/status` (Discord command) - Bot health
- Celery health check in docker-compose

### Logging

- `logs/bot.log` - Discord bot logs
- `logs/api.log` - FastAPI logs
- `logs/celery.log` - Celery worker logs
- `logs/celery-beat.log` - Celery beat logs

### Metrics

- Database query performance
- Discord API latency
- Webhook processing time
- Celery task queue length
- Redis memory usage
