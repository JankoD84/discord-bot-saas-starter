# Installation Guide

## Overview & Time Estimate

**Total time:** 15-30 minutes (depending on experience)

| Step | Time | What you'll do |
|---|---|---|
| **1. Prerequisites** | 5-10 min | Install Python, Git, Docker (optional) |
| **2. Discord Bot Setup** | 5 min | Create bot in Discord Developer Portal |
| **3. Code Setup** | 5-10 min | Clone repo, install dependencies |
| **4. Configuration** | 2-5 min | Set up environment variables |
| **5. First Run** | 2 min | Start bot and test commands |

**📍 Need help?** See [Troubleshooting Guide](TROUBLESHOOTING.md) for common issues.

---

## System Requirements

### Minimum Requirements
- **Operating System**: Linux, macOS, or Windows 10+
- **Python**: Version 3.10 or higher
- **Memory**: 512MB RAM (1GB recommended)
- **Storage**: 100MB free space
- **Network**: Internet connection for Discord API

### Optional Requirements
- **Docker**: Version 20.10+ (for containerized deployment)
- **Redis**: Version 6.0+ (for background tasks and rate limiting)
- **Domain**: Custom domain (for production webhooks)

## 🔧 Prerequisites Installation

### Install Python 3.10+

#### Ubuntu/Debian
```bash
sudo apt update
sudo apt install python3.10 python3.10-venv python3.10-pip
```

#### CentOS/RHEL
```bash
sudo yum install python310 python310-pip
```

#### macOS
```bash
# Using Homebrew
brew install python@3.10

# Or download from python.org
```

#### Windows
1. Download from [python.org](https://www.python.org/downloads/)
2. Run installer and check "Add to PATH"

### Install Git (Required)
```bash
# Ubuntu/Debian
sudo apt install git

# macOS
brew install git

# Windows
# Download from git-scm.com
```

### Install Docker (Optional)
```bash
# Ubuntu
sudo apt install docker.io docker-compose

# macOS
# Download Docker Desktop from docker.com

# Windows
# Download Docker Desktop from docker.com
```

## 📥 Download and Setup

### 1. Clone Repository
```bash
git clone https://github.com/YOUR_USERNAME/discord-bot-starter-kit.git
# Note: replace YOUR_USERNAME with your GitHub username,
# or skip this step if you downloaded the ZIP directly
cd discord-bot-starter-kit
```

### 2. Create Virtual Environment
```bash
# Create virtual environment
python3.10 -m venv venv

# Activate (Linux/macOS)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## 🤖 Discord Bot Setup

### 1. Create Discord Application
1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **"New Application"**
3. Enter application name (e.g., "My Bot")
4. Click **"Create"**

### 2. Create Bot User
1. In your application, go to **"Bot"** tab
2. Click **"Add Bot"**
3. Confirm with **"Yes, do it!"**

### 3. Configure Bot Permissions
Under **"Privileged Gateway Intents"**, enable:
- ✅ **SERVER MEMBERS INTENT**
- ✅ **MESSAGE CONTENT INTENT**

Click **"Save Changes"**

### 4. Copy Bot Token
1. Still in **"Bot"** tab
2. Click **"Reset Token"** (if first time) or **"Copy"**
3. **IMPORTANT**: Keep this token secure - never share it!

### 5. Generate Invite Link
1. Go to **"OAuth2"** → **"URL Generator"**
2. Select scopes:
   - ✅ `bot`
   - ✅ `applications.commands`
3. Select bot permissions:
   - ✅ Send Messages
   - ✅ Embed Links
   - ✅ Use Slash Commands
   - ✅ Read Message History
4. Copy the generated URL
5. Paste in browser and invite to your server

### 6. Get Server ID (Optional)
1. In Discord: User Settings → Advanced
2. Enable **Developer Mode**
3. Right-click your server → **"Copy Server ID"**

## ⚙️ Environment Configuration

### 1. Create Environment File
```bash
# For development
cp .env.example .env.dev

# For production
cp .env.example .env.prod
```

### 2. Edit Environment File

#### Minimum Configuration (Demo Mode)
Edit `.env.dev`:
```env
# Environment
APP_ENV=dev

# Discord Bot (REQUIRED)
DISCORD_BOT_TOKEN_DEV=your_bot_token_here
DISCORD_GUILD_ID_DEV=your_server_id_here  # Optional

# Features
FEATURE_DEMO_MODE=true
FEATURE_PAYMENTS=false
FEATURE_ADMIN=true
```

#### Full Configuration (With Payments)
Edit `.env.dev` or `.env.prod`:
```env
# Environment
APP_ENV=dev  # or prod

# Discord Bot
DISCORD_BOT_TOKEN_DEV=your_bot_token_here
DISCORD_GUILD_ID_DEV=your_server_id_here

# Features
FEATURE_DEMO_MODE=false
FEATURE_PAYMENTS=true
FEATURE_ADMIN=true

# Stripe (if using payments)
STRIPE_SECRET_KEY_DEV=sk_test_your_test_key_here
STRIPE_PRICE_ID_DEV=price_your_price_id_here
STRIPE_WEBHOOK_SECRET_DEV=whsec_your_webhook_secret_here
APP_BASE_URL_DEV=http://localhost:8000

# Admin (comma-separated Discord user IDs)
ADMIN_USER_IDS=123456789,987654321

# Bot Branding (optional)
BOT_NAME=My Bot
PREMIUM_BRAND_NAME=My Bot Premium
```

### 3. Validate Configuration
```bash
python scripts/check_env.py
```

This script will:
- ✅ Check Discord token format
- ✅ Validate Stripe configuration (if enabled)
- ✅ Test database permissions
- ✅ Check Redis connection (if configured)
- ✅ Provide helpful error messages

## 🚀 First Run

### Option 1: Development Mode
```bash
# Using the run script
./scripts/run_dev.sh

# Or manually
source venv/bin/activate
python -m src.app.discord_bot_app
```

### Option 2: Production Mode
```bash
# Using the run script
./scripts/run_prod.sh

# Or with environment file
ENV_FILE=/path/to/.env.prod python -m src.app.discord_bot_app
```

### Option 3: Docker (Recommended for Production)
```bash
# Build and start all services
docker-compose up -d

# Check logs
docker-compose logs -f discord-bot
```

## ✅ Verify Installation

### 1. Check Bot is Running
In your Discord server, type:
```
/ping
```
You should see a bot status response.

### 2. Test Demo Mode
```
/demo
```
This should start an interactive demo flow.

### 3. Check System Status
```
/status
```
Shows database connection, features enabled, etc.

## 🔧 Common Setup Issues

### Bot Token Not Working
- ✅ Token is 50+ characters long
- ✅ No extra spaces or line breaks
- ✅ Bot has correct intents enabled
- ✅ Bot was invited to server with proper permissions

### Commands Not Appearing
- ✅ Wait 1-5 minutes for commands to register
- ✅ Try reinviting the bot
- ✅ Check if guild ID is set (faster registration)

### Database Errors
- ✅ Check write permissions to `./data/` directory
- ✅ Ensure DB_PATH is correct
- ✅ Try deleting and recreating database

### Stripe Issues
- ✅ Using test keys in dev environment
- ✅ Webhook URL is accessible
- ✅ Price ID exists in Stripe

## 📦 Production Deployment

### Docker Deployment (Recommended)
```bash
# 1. Configure production environment
cp .env.example .env.prod
# Edit .env.prod with production values

# 2. Deploy
docker-compose -f docker-compose.yml -f compose.prod.yml up -d

# 3. Setup SSL
# Place certificates in nginx/ssl/
```

### Manual VPS Deployment
```bash
# 1. Create system user
sudo useradd -m -s /bin/bash botuser

# 2. Setup directory
sudo mkdir -p /opt/discord-bot
sudo chown botuser:botuser /opt/discord-bot

# 3. Clone and setup
sudo -u botuser git clone <repo> /opt/discord-bot
cd /opt/discord-bot
sudo -u botuser python3 -m venv venv
sudo -u botuser venv/bin/pip install -r requirements.txt

# 4. Create secure env file
sudo mkdir -p /etc/discord-bot
sudo nano /etc/discord-bot/prod.env
sudo chmod 600 /etc/discord-bot/prod.env

# 5. Create systemd service
sudo cp deploy/systemd/discord-bot.service /etc/systemd/system/
sudo systemctl enable discord-bot
sudo systemctl start discord-bot
```

## 🎯 Next Steps

1. **Customize bot name and branding**
2. **Add your own commands**
3. **Set up Stripe for real payments**
4. **Configure domain and SSL**
5. **Set up monitoring and backups**

## 📞 Getting Help

- **Run diagnostics**: `python scripts/check_env.py`
- **Check logs**: `logs/bot.log`
- **Test commands**: `/ping`, `/status`
- **Review docs**: Check `docs/` directory
- **Support**: See [Troubleshooting Guide](TROUBLESHOOTING.md) or open an issue on GitHub

---

**Installation complete!** Your Discord bot is now ready for customization.
