# Environment Variables Reference

Complete reference for all configuration variables in the Discord Bot Starter Kit.

## Environment Files

- `.env.example` - Template with all variables and documentation
- `.env.dev` - Development environment configuration
- `.env.prod` - Production environment configuration

**Important:** Never commit `.env.dev` or `.env.prod` to version control. These files contain secrets.

## Required Variables

### APP_ENV

**Required:** Yes  
**Values:** `dev` | `prod`  
**Default:** None

Sets the operating environment. This variable controls which configuration variants are loaded (DEV vs PROD variants for other variables).

```env
APP_ENV=dev
```

### DISCORD_BOT_TOKEN_DEV / DISCORD_BOT_TOKEN_PROD

**Required:** Yes (at least one)  
**Values:** Discord bot token string (50+ characters)  
**Default:** None

Discord bot token from Discord Developer Portal. Required for the bot to connect to Discord.

- `DISCORD_BOT_TOKEN_DEV` - Used when `APP_ENV=dev`
- `DISCORD_BOT_TOKEN_PROD` - Used when `APP_ENV=prod`

```env
DISCORD_BOT_TOKEN_DEV=MTEyMzQ1Njc4OTAxMjM0NTY3ODkwMTIzNDU2Nzg5MA.GhIjKl.MnOpQrStUvWxYzAbCdEfGhIjKlMnOpQrStUvWxYzAbCdEf
```

**How to obtain:**
1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Select your application → Bot → Reset Token
3. Copy the token immediately (it's only shown once)

## Optional Variables

### Discord Configuration

#### DISCORD_GUILD_ID_DEV / DISCORD_GUILD_ID_PROD

**Required:** No  
**Values:** Numeric Discord guild ID  
**Default:** None

Optional Discord server ID for faster command registration in development. When set, slash commands sync to this guild immediately (1-5 seconds). When not set, commands sync globally (up to 1 hour).

```env
DISCORD_GUILD_ID_DEV=123456789012345678
```

**How to obtain:**
1. Enable Developer Mode in Discord (User Settings → Advanced)
2. Right-click your server → Copy Server ID

#### DISCORD_CLIENT_ID_DEV / DISCORD_CLIENT_ID_PROD

**Required:** No (only for SaaS OAuth integration)  
**Values:** Discord application client ID  
**Default:** None

Discord OAuth2 client ID for SaaS dashboard integration. Required if using Discord OAuth for tenant authentication.

```env
DISCORD_CLIENT_ID_DEV=123456789012345678
```

#### DISCORD_CLIENT_SECRET_DEV / DISCORD_CLIENT_SECRET_PROD

**Required:** No (only for SaaS OAuth integration)  
**Values:** Discord OAuth2 client secret  
**Default:** None

Discord OAuth2 client secret. Required if using Discord OAuth for tenant authentication.

```env
DISCORD_CLIENT_SECRET_DEV=abc123def456ghi789jkl012mno345pq
```

#### DISCORD_REDIRECT_URI_DEV / DISCORD_REDIRECT_URI_PROD

**Required:** No (only for SaaS OAuth integration)  
**Values:** Full URL  
**Default:** None

OAuth redirect URI for Discord OAuth. Must point to your frontend URL (SPA), not backend callback.

```env
DISCORD_REDIRECT_URI_DEV=http://localhost:8000/
DISCORD_REDIRECT_URI_PROD=https://your-bot-domain.com/
```

### Feature Flags

#### FEATURE_PAYMENTS

**Required:** No  
**Values:** `true` | `false`  
**Default:** `true`

Enable or disable all payment-related features (Stripe integration, premium commands). When `false`, Stripe is not initialized and payment commands are disabled.

```env
FEATURE_PAYMENTS=true
```

#### FEATURE_ADMIN

**Required:** No  
**Values:** `true` | `false`  
**Default:** `true`

Enable or disable admin commands (`/admin_stats`). When `false`, admin commands return "Currently Unavailable" for all users.

```env
FEATURE_ADMIN=true
```

#### FEATURE_DEMO_MODE

**Required:** No  
**Values:** `true` | `false`  
**Default:** `false`

Enable demo mode for testing and development. When `true`, the bot simulates premium functionality without Stripe integration. Demo state persists in database.

```env
FEATURE_DEMO_MODE=true
```

### Database Configuration

#### DATABASE_TYPE

**Required:** No  
**Values:** `sqlite` | `postgresql`  
**Default:** `sqlite`

Database backend selection. Use `sqlite` for development and small deployments. Use `postgresql` for production with high concurrency requirements.

```env
DATABASE_TYPE=sqlite
```

#### DB_PATH_DEV / DB_PATH_PROD

**Required:** No (only when DATABASE_TYPE=sqlite)  
**Values:** File path  
**Default:** `./data/bot_dev.db` / `./data/bot_prod.db`

SQLite database file path. Separate databases per environment recommended.

```env
DB_PATH_DEV=./data/bot_dev.db
DB_PATH_PROD=/var/lib/discord-bot/bot_prod.db
```

#### POSTGRES_HOST

**Required:** Yes (when DATABASE_TYPE=postgresql)  
**Values:** hostname or IP  
**Default:** `localhost`

PostgreSQL database host.

```env
POSTGRES_HOST=localhost
```

#### POSTGRES_PORT

**Required:** No  
**Values:** Port number  
**Default:** `5432`

PostgreSQL database port.

```env
POSTGRES_PORT=5432
```

#### POSTGRES_USER

**Required:** Yes (when DATABASE_TYPE=postgresql)  
**Values:** Username  
**Default:** `discordbot`

PostgreSQL database user.

```env
POSTGRES_USER=discordbot
```

#### POSTGRES_PASSWORD

**Required:** Yes (when DATABASE_TYPE=postgresql)  
**Values:** Password  
**Default:** `discordbot`

PostgreSQL database password.

```env
POSTGRES_PASSWORD=secure_password_here
```

#### POSTGRES_DB

**Required:** Yes (when DATABASE_TYPE=postgresql)  
**Values:** Database name  
**Default:** `discordbot`

PostgreSQL database name.

```env
POSTGRES_DB=discordbot
```

### Stripe Configuration

#### STRIPE_SECRET_KEY_DEV / STRIPE_SECRET_KEY_PROD

**Required:** Yes (when FEATURE_PAYMENTS=true)  
**Values:** Stripe API key  
**Default:** None

Stripe secret key for API access. Must match environment (test keys for dev, live keys for prod). The application validates key prefixes and will fail startup if keys don't match environment.

- Dev: Must start with `sk_test_...`
- Prod: Must start with `sk_live_...`

```env
STRIPE_SECRET_KEY_DEV=sk_test_51AbCdEfGhIjKlMnOpQrStUvWxYzAbCdEfGhIjKlMnOpQrStUvWxYzAbCdEf
```

**Validation:** The application enforces test/live key matching to prevent accidental use of production keys in development.

#### STRIPE_PRICE_ID_DEV / STRIPE_PRICE_ID_PROD

**Required:** Yes (when FEATURE_PAYMENTS=true)  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for the default subscription plan. This is the price ID used in the `/pricing` command.

```env
STRIPE_PRICE_ID_DEV=price_1AbCdEfGhIjKlMnOpQrStUvW
```

**How to obtain:**
1. Stripe Dashboard → Products → Pricing
2. Create or select a price
3. Copy the Price ID (starts with `price_`)

#### STRIPE_WEBHOOK_SECRET_DEV / STRIPE_WEBHOOK_SECRET_PROD

**Required:** Yes (when FEATURE_PAYMENTS=true)  
**Values:** Stripe webhook signing secret  
**Default:** None

Stripe webhook signing secret for signature verification. Required for secure webhook processing.

```env
STRIPE_WEBHOOK_SECRET_DEV=whsec_abc123def456ghi789jkl012mno345pq
```

**How to obtain:**
1. Stripe Dashboard → Developers → Webhooks
2. Add endpoint with your URL
3. Copy the signing secret (starts with `whsec_`)

#### STRIPE_WEBHOOK_SECRET_DISCORD_DEV / STRIPE_WEBHOOK_SECRET_DISCORD_PROD

**Required:** No (legacy fallback available)  
**Values:** Stripe webhook signing secret  
**Default:** None

Discord-specific webhook secret for `/webhooks/stripe/webhook` endpoint. If not set, falls back to `STRIPE_WEBHOOK_SECRET_*`.

```env
STRIPE_WEBHOOK_SECRET_DISCORD_DEV=whsec_abc123def456ghi789jkl012mno345pq
```

#### STRIPE_WEBHOOK_SECRET_SAAS_DEV / STRIPE_WEBHOOK_SECRET_SAAS_PROD

**Required:** No (legacy fallback available)  
**Values:** Stripe webhook signing secret  
**Default:** None

SaaS-specific webhook secret for `/webhook` endpoint. If not set, falls back to `STRIPE_WEBHOOK_SECRET_*`.

```env
STRIPE_WEBHOOK_SECRET_SAAS_DEV=whsec_xyz789ghi012jkl345mno678pqr901stu
```

### Pricing Plan Configuration

#### STRIPE_PRICE_ID_MONTHLY_DEV / STRIPE_PRICE_ID_MONTHLY_PROD

**Required:** No  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for monthly subscription plan.

```env
STRIPE_PRICE_ID_MONTHLY_DEV=price_1AbCdEfGhIjKlMnOpQrStUvW
```

#### STRIPE_PRICE_ID_YEARLY_DEV / STRIPE_PRICE_ID_YEARLY_PROD

**Required:** No  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for yearly subscription plan.

```env
STRIPE_PRICE_ID_YEARLY_DEV=price_1XyZwVuTsRqPoNmLkJiHgFeDcBa
```

#### STRIPE_PRICE_ID_LIFETIME_DEV / STRIPE_PRICE_ID_LIFETIME_PROD

**Required:** No  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for lifetime (one-time) purchase.

```env
STRIPE_PRICE_ID_LIFETIME_DEV=price_1LkJiHgFeDcBaZyXwVuTsRqPoNm
```

### SaaS Billing Configuration

#### STRIPE_SAAS_PRICE_STARTER_MONTHLY_DEV / STRIPE_SAAS_PRICE_STARTER_MONTHLY_PROD

**Required:** No  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for SaaS Starter monthly plan (tenant billing).

```env
STRIPE_SAAS_PRICE_STARTER_MONTHLY_DEV=price_1AbCdEfGhIjKlMnOpQrStUvW
```

#### STRIPE_SAAS_PRICE_PRO_MONTHLY_DEV / STRIPE_SAAS_PRICE_PRO_MONTHLY_PROD

**Required:** No  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for SaaS Pro monthly plan (tenant billing).

```env
STRIPE_SAAS_PRICE_PRO_MONTHLY_DEV=price_1XyZwVuTsRqPoNmLkJiHgFeDcBa
```

#### STRIPE_SAAS_PRICE_ENTERPRISE_MONTHLY_DEV / STRIPE_SAAS_PRICE_ENTERPRISE_MONTHLY_PROD

**Required:** No  
**Values:** Stripe Price ID  
**Default:** None

Stripe Price ID for SaaS Enterprise monthly plan (tenant billing).

```env
STRIPE_SAAS_PRICE_ENTERPRISE_MONTHLY_DEV=price_1LkJiHgFeDcBaZyXwVuTsRqPoNm
```

**Note:** Plans without configured Price IDs are excluded from availability in the dashboard.

### Application Configuration

#### APP_BASE_URL_DEV / APP_BASE_URL_PROD

**Required:** Yes (when using webhooks or OAuth)  
**Values:** Full URL  
**Default:** `http://localhost:8000` (dev)

Base URL for the application. Used for:
- Webhook URLs in Stripe checkout sessions
- OAuth redirect URIs
- Email links (password reset, verification)

```env
APP_BASE_URL_DEV=http://localhost:8000
APP_BASE_URL_PROD=https://your-bot-domain.com
```

**Important:** Production must use HTTPS for Stripe webhooks to work correctly.

#### BOT_EXAMPLE

**Required:** No  
**Values:** String  
**Default:** `discord_bot`

Example bot identifier. Used for framework-specific configuration loading.

```env
BOT_EXAMPLE=discord_bot
```

### CORS Configuration

#### CORS_ALLOWED_ORIGINS_DEV / CORS_ALLOWED_ORIGINS_PROD

**Required:** No  
**Values:** Comma-separated list of URLs  
**Default:** `http://localhost:8000,http://localhost:3000` (dev)

Allowed origins for Cross-Origin Resource Sharing (CORS). In production, this must be explicitly configured (defaults to empty for security).

```env
CORS_ALLOWED_ORIGINS_DEV=http://localhost:8000,http://localhost:3000
CORS_ALLOWED_ORIGINS_PROD=https://your-bot-domain.com,https://app.your-bot-domain.com
```

### Redis Configuration

#### REDIS_URL

**Required:** No  
**Values:** Redis connection URL  
**Default:** None

Full Redis connection URL. Takes precedence over individual REDIS_* settings.

```env
REDIS_URL=redis://localhost:6379/0
```

#### REDIS_HOST

**Required:** No  
**Values:** hostname or IP  
**Default:** `localhost`

Redis server host.

```env
REDIS_HOST=localhost
```

#### REDIS_PORT

**Required:** No  
**Values:** Port number  
**Default:** `6379`

Redis server port.

```env
REDIS_PORT=6379
```

#### REDIS_PASSWORD

**Required:** No  
**Values:** Password  
**Default:** None

Redis server password. Leave empty if no authentication required.

```env
REDIS_PASSWORD=your_redis_password
```

#### REDIS_DB

**Required:** No  
**Values:** Database number  
**Default**: `0`

Redis database number.

```env
REDIS_DB=0
```

### Authentication Configuration

#### JWT_SECRET_KEY

**Required:** Yes (in production)  
**Values:** Secure random string  
**Default:** Hardcoded dev-only fallback (with warning)

JWT secret key for signing and verifying authentication tokens. Must be set to a strong random value in production. Startup will fail if unset or still set to placeholder in production.

```bash
# Generate secure value
JWT_SECRET_KEY=$(openssl rand -hex 32)
```

```env
JWT_SECRET_KEY=abc123def4567890123456789012345678901234567890123456789012345678
```

**Security:** Never use the example value in production. Rotate this key periodically.

#### INTERNAL_ADMIN_TOKEN

**Required:** No  
**Values:** Secure random string  
**Default:** None

Internal admin API token for diagnostics endpoints at `/internal/tenant/{id}/summary` and `/internal/tenant/{id}/diagnostics`. Protects internal endpoints from unauthorized access.

```bash
# Generate secure token
INTERNAL_ADMIN_TOKEN=$(openssl rand -base64 32)
```

```env
INTERNAL_ADMIN_TOKEN=abc123def4567890123456789012345678901234567890123456789012345678
```

### Admin Configuration

#### ADMIN_USER_IDS

**Required:** No  
**Values:** Comma-separated Discord user IDs  
**Default:** None

Discord user IDs allowed to access admin commands (`/admin_stats`). Users not in this list cannot access admin functionality.

```env
ADMIN_USER_IDS=123456789012345678,987654321098765432
```

**How to obtain:**
1. Enable Developer Mode in Discord (User Settings → Advanced)
2. Right-click your username → Copy User ID

### Bot Branding Configuration

#### BOT_NAME

**Required:** No  
**Values:** String  
**Default:** None

Bot display name used in messages and UI.

```env
BOT_NAME=My Awesome Bot
```

#### PREMIUM_BRAND_NAME

**Required:** No  
**Values:** String  
**Default:** None

Brand name for premium features.

```env
PREMIUM_BRAND_NAME=My Awesome Bot Pro
```

#### BOT_WELCOME_MESSAGE

**Required:** No  
**Values:** String  
**Default:** None

Welcome message shown to users.

```env
BOT_WELCOME_MESSAGE=Welcome! Type /ping to check if the bot is running.
```

### Framework Configuration

#### DEFAULT_LANGUAGE

**Required:** No  
**Values:** Language code  
**Default:** `en`

Default language for the bot. Supported codes: `en`, `sk`, `cs`, `de`, `fr`, `es`.

```env
DEFAULT_LANGUAGE=en
```

#### SUPPORTED_LANGUAGES

**Required:** No  
**Values:** Comma-separated language codes  
**Default:** `en,sk,cs,de,fr,es`

List of supported languages.

```env
SUPPORTED_LANGUAGES=en,sk,cs,de,fr,es
```

### Email/SMTP Configuration

#### SMTP_HOST

**Required:** No  
**Values:** SMTP server hostname  
**Default:** None

SMTP server host for email sending. If not set, email links are logged as warnings instead (dev mode).

```env
SMTP_HOST=smtp.mailgun.org
```

#### SMTP_PORT

**Required:** No (if SMTP_HOST set)  
**Values:** Port number  
**Default:** `587` (STARTTLS)

SMTP server port. Use `587` for STARTTLS, `465` for implicit TLS.

```env
SMTP_PORT=587
```

#### SMTP_USER

**Required:** No (if SMTP_HOST set)  
**Values:** SMTP username  
**Default:** None

SMTP authentication username.

```env
SMTP_USER=postmaster@mg.yourdomain.com
```

#### SMTP_PASSWORD

**Required:** No (if SMTP_HOST set)  
**Values:** SMTP password  
**Default:** None

SMTP authentication password.

```env
SMTP_PASSWORD=your-smtp-api-key-or-password
```

#### EMAIL_FROM

**Required:** No (if SMTP_HOST set)  
**Values:** Email address  
**Default:** None

From address for outgoing emails.

```env
EMAIL_FROM=noreply@yourdomain.com
```

**Supported providers:** Mailgun, SendGrid SMTP, Postmark, AWS SES, Gmail.

### Logging Configuration

#### LOG_LEVEL

**Required:** No  
**Values:** `DEBUG` | `INFO` | `WARNING` | `ERROR`  
**Default:** `INFO`

Logging verbosity level.

```env
LOG_LEVEL=INFO
```

#### LOG_FILE

**Required:** No  
**Values**: File path  
**Default:** `./logs/bot.log`

Log file path.

```env
LOG_FILE=/var/log/discord-bot/bot.log
```

#### LOG_FORMAT

**Required:** No  
**Values**: `json` | `text`  
**Default**: `json`

Log format. JSON format recommended for production log aggregation.

```env
LOG_FORMAT=json
```

## Environment Validation

Run the validation script to check your configuration:

```bash
python scripts/check_env.py
```

This script validates:
- Discord token format (50+ characters)
- Stripe key prefixes (test vs live)
- Database directory permissions
- Optional configuration with warnings
- Helpful error messages for misconfiguration

## Security Best Practices

1. **Never commit .env files** to version control
2. **Use separate tokens** for dev and prod environments
3. **Rotate secrets** periodically (quarterly recommended)
4. **Set file permissions** to 600 on production env files
5. **Use strong random values** for JWT_SECRET_KEY and INTERNAL_ADMIN_TOKEN
6. **Validate test vs live keys** - the application enforces this
7. **Store production secrets** in secure locations (e.g., `/etc/discord-bot/prod.env`)

## Common Configuration Patterns

### Development (Demo Mode)

```env
APP_ENV=dev
DISCORD_BOT_TOKEN_DEV=your_dev_token
FEATURE_DEMO_MODE=true
FEATURE_PAYMENTS=false
```

### Development (Stripe Test Mode)

```env
APP_ENV=dev
DISCORD_BOT_TOKEN_DEV=your_dev_token
FEATURE_DEMO_MODE=false
FEATURE_PAYMENTS=true
STRIPE_SECRET_KEY_DEV=sk_test_...
STRIPE_PRICE_ID_DEV=price_...
STRIPE_WEBHOOK_SECRET_DEV=whsec_...
APP_BASE_URL_DEV=http://localhost:8000
```

### Production (Payments)

```env
APP_ENV=prod
DISCORD_BOT_TOKEN_PROD=your_prod_token
FEATURE_DEMO_MODE=false
FEATURE_PAYMENTS=true
STRIPE_SECRET_KEY_PROD=sk_live_...
STRIPE_PRICE_ID_PROD=price_...
STRIPE_WEBHOOK_SECRET_PROD=whsec_...
APP_BASE_URL_PROD=https://your-bot-domain.com
JWT_SECRET_KEY=your_jwt_secret
DATABASE_TYPE=postgresql
POSTGRES_HOST=your-db-host
POSTGRES_USER=discordbot
POSTGRES_PASSWORD=your-db-password
POSTGRES_DB=discordbot
```

### Production (Full SaaS)

```env
APP_ENV=prod
DISCORD_BOT_TOKEN_PROD=your_prod_token
FEATURE_DEMO_MODE=false
FEATURE_PAYMENTS=true
STRIPE_SECRET_KEY_PROD=sk_live_...
STRIPE_PRICE_ID_PROD=price_...
STRIPE_WEBHOOK_SECRET_PROD=whsec_...
STRIPE_WEBHOOK_SECRET_DISCORD_PROD=whsec_...
STRIPE_WEBHOOK_SECRET_SAAS_PROD=whsec_...
APP_BASE_URL_PROD=https://your-bot-domain.com
JWT_SECRET_KEY=your_jwt_secret
INTERNAL_ADMIN_TOKEN=your_admin_token
DATABASE_TYPE=postgresql
POSTGRES_HOST=your-db-host
POSTGRES_USER=discordbot
POSTGRES_PASSWORD=your-db-password
POSTGRES_DB=discordbot
REDIS_URL=redis://your-redis-host:6379/0
SMTP_HOST=smtp.mailgun.org
SMTP_PORT=587
SMTP_USER=postmaster@mg.yourdomain.com
SMTP_PASSWORD=your-smtp-password
EMAIL_FROM=noreply@yourdomain.com
```

## Troubleshooting

### "STRIPE_SECRET_KEY not found"

Ensure `FEATURE_PAYMENTS=true` and the appropriate `STRIPE_SECRET_KEY_*` variable is set for your environment.

### "Using test key in prod environment"

Ensure `STRIPE_SECRET_KEY_PROD` starts with `sk_live_...` and `STRIPE_SECRET_KEY_DEV` starts with `sk_test_...`.

### "Database directory is not writable"

Ensure the directory specified in `DB_PATH_*` exists and has write permissions: `chmod 755 ./data`.

### "JWT_SECRET_KEY not set in production"

Generate a secure JWT secret: `openssl rand -hex 32` and set `JWT_SECRET_KEY` in your production env file.
