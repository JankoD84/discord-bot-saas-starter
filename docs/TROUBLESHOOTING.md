# Troubleshooting Guide

## Quick Reference

| ❌ Problem | 🔧 Solution | 📖 Details |
|---|---|---|
| **Bot won't start** | Run `python scripts/check_env.py` | [Bot Won't Start](#bot-wont-start) |
| **Commands not working** | Check permissions & wait 1-5 min | [Commands Not Working](#commands-not-working) |
| **Database errors** | Check `./data/` permissions | [Database Issues](#database-issues) |
| **Payments failing** | Verify Stripe keys & webhooks | [Payment/Stripe Issues](#paymentstripe-issues) |
| **Docker issues** | Check logs with `docker-compose logs` | [Docker Issues](#docker-issues) |

**🚨 Emergency:** Bot completely down? See [Emergency Recovery](#-emergency-recovery)

---

## 🚨 Quick Diagnosis

Start with these commands to quickly identify issues:
```bash
# 1. Check environment
python scripts/check_env.py

# 2. Test basic bot functionality
python -c "from src.app.discord_bot_app import create_bot; print('Bot creation OK')"

# 3. Check database
python -c "from src.app.db.database import init_db; init_db(); print('Database OK')"

# 4. View recent logs
tail -f logs/bot.log
```

## 🔧 Common Issues

### Bot Won't Start

#### Error: "DISCORD_BOT_TOKEN not found"
```
❌ DISCORD_BOT_TOKEN not found
💡 Set DISCORD_BOT_TOKEN_DEV (for dev) or DISCORD_BOT_TOKEN_PROD (for prod)
```

**Solution:**
1. Copy environment file: `cp .env.example .env.dev`
2. Edit `.env.dev` and add your bot token:
   ```env
   DISCORD_BOT_TOKEN_DEV=your_actual_bot_token_here
   ```
3. Token must be 50+ characters from Discord Developer Portal

#### Error: "Invalid token format"
```
❌ Invalid token format
💡 Discord tokens should be 50+ characters long
```

**Solution:**
- Ensure you copied the full token without extra spaces
- Get a fresh token from Discord Developer Portal → Bot → Reset Token

#### Error: "Privileged Intents Required"
```
discord.errors.PrivilegedIntentsRequired: At least one privileged intent is required
```

**Solution:**
1. Go to Discord Developer Portal
2. Your Application → Bot → Privileged Gateway Intents
3. Enable: ✅ Message Content Intent, ✅ Server Members Intent

### Commands Not Working

#### Commands not appearing in Discord
**Symptoms:**
- `/ping` shows no suggestions
- "Interaction failed" errors

**Solutions:**
1. **Wait for sync**: Commands can take 1-5 minutes to appear
2. **Check guild ID** (development only):
   ```env
   DISCORD_GUILD_ID_DEV=your_server_id_here
   ```
3. **Reinvite the bot**:
   - Revoke bot permissions
   - Use fresh invite URL with correct scopes
4. **Check bot status**:
   ```bash
   python -m src.app.discord_bot_app
   # Look for: "Registered X slash commands"
   ```

#### "Interaction failed" error
**Causes:**
- Bot restarted while command was executing
- Network timeout
- Missing permissions

**Solutions:**
1. Try the command again
2. Check bot has "Send Messages" permission
3. Check logs for errors: `tail -f logs/bot.log`

### Database Issues

#### Error: "Database directory is not writable"
```
❌ Database directory is not writable
💡 Check permissions for: ./data
```

**Solution:**
```bash
# Create data directory
mkdir -p ./data

# Set permissions
chmod 755 ./data

# Or use absolute path in .env
DB_PATH_DEV=/tmp/bot.db
```

#### Error: "no such table: users"
**Solution:**
- Database initializes automatically on first run
- If corrupted, delete and restart:
  ```bash
  rm ./data/bot_*.db
  python -m src.app.discord_bot_app
  ```

#### Database locked error
**Symptoms:**
- "database is locked" errors
- Commands not saving data

**Solutions:**
1. Check for other bot instances:
   ```bash
   ps aux | grep discord_bot_app
   ```
2. Kill existing processes:
   ```bash
   pkill -f discord_bot_app
   ```
3. Remove lock file:
   ```bash
   rm ./data/bot_*.db-journal
   ```

### Payment/Stripe Issues

#### Error: "No Stripe configuration found"
```
❌ STRIPE_SECRET_KEY not found
💡 Set STRIPE_SECRET_KEY_DEV (dev) or STRIPE_SECRET_KEY_PROD (prod)
```

**Solution:**
1. Enable payments in environment:
   ```env
   FEATURE_PAYMENTS=true
   ```
2. Add Stripe configuration:
   ```env
   STRIPE_SECRET_KEY_DEV=sk_test_your_test_key_here
   STRIPE_PRICE_ID_DEV=price_your_price_id_here
   STRIPE_WEBHOOK_SECRET_DEV=whsec_your_webhook_secret_here
   ```

#### Webhook not receiving events
**Symptoms:**
- Payments complete but premium not activated
- No webhook logs

**Solutions:**
1. **Check webhook URL** in Stripe Dashboard:
   - Must be: `https://your-domain.com/webhooks/stripe/webhook`
2. **Test with Stripe CLI**:
   ```bash
   stripe listen --forward-to localhost:8000/webhooks/stripe/webhook
   stripe trigger checkout.session.completed
   ```
3. **Check webhook secret** matches exactly
4. **Verify firewall** allows port 8000

#### Test vs Live Keys
**Error: Using test key in production**
```bash
# Check which key you're using
grep STRIPE_SECRET_KEY .env.prod
```

**Fix:**
- Dev: Use `sk_test_...` keys
- Prod: Use `sk_live_...` keys
- Never mix them!

### Docker Issues

#### Container won't start
```bash
# Check logs
docker-compose logs discord-bot

# Common issues:
# - Missing .env file
# - Wrong permissions
# - Port conflicts
```

#### Error: "port already in use"
```bash
# Find what's using the port
sudo netstat -tulpn | grep :8000

# Kill the process
sudo kill -9 <PID>

# Or use different port
docker-compose down
docker-compose up -d --scale api=1
```

#### Database persistence issues
**Symptoms:**
- Data lost on container restart
- Database resets to empty

**Solution:**
Check volume mounts in `docker-compose.yml`:
```yaml
volumes:
  - ./data:/app/data  # Must be mapped
```

### Performance Issues

#### Bot slow to respond
**Check:**
1. **Discord API latency**:
   ```bash
   # In Discord: /ping
   # Look at latency value
   ```
2. **Database size**:
   ```bash
   ls -lh ./data/bot_*.db
   # Consider cleanup if >100MB
   ```
3. **Memory usage**:
   ```bash
   free -h
   ps aux | grep python
   ```

#### Memory leak
**Symptoms:**
- Memory usage increases over time
- Bot crashes after hours

**Solutions:**
1. Restart bot regularly:
   ```bash
   # In systemd: Restart=always
   # Or cron: 0 4 * * * systemctl restart discord-bot
   ```
2. Check for unclosed connections
3. Monitor with `htop`

### SSL/HTTPS Issues

#### Certificate errors
```bash
# Check certificate expiry
openssl x509 -in /etc/letsencrypt/live/your-domain.com/cert.pem -text -noout

# Renew if needed
certbot renew
```

#### nginx configuration errors
```bash
# Test nginx config
nginx -t

# Reload after changes
nginx -s reload

# Check error log
tail -f /var/log/nginx/error.log
```

## 🔍 Debug Mode

### Enable Debug Logging
```env
LOG_LEVEL=DEBUG
```

### Common Debug Commands
```bash
# Test Discord connection
python -c "
import discord
from src.app.config import get_discord_bot_token
intents = discord.Intents.default()
client = discord.Client(intents=intents)
@client.event
async def on_ready():
    print(f'Connected as {client.user}')
    await client.close()
client.run(get_discord_bot_token())
"

# Test database
python -c "
from src.app.db.database import init_db, get_basic_stats
init_db()
print(get_basic_stats())
"

# Test Stripe
python -c "
from src.app.config import get_stripe_settings
from src.app.payments.stripe_service import init_stripe
settings = get_stripe_settings()
init_stripe(settings.secret_key)
print('Stripe OK')
"
```

## 📊 Health Monitoring

### Built-in Health Checks
```bash
# In Discord: /status
# Shows:
# - Bot connection status
# - Database connection
# - Feature flags
# - Uptime
```

### System Monitoring Script
Create `monitor.sh`:
```bash
#!/bin/bash
echo "=== Bot Health Check ==="
# Check if bot is running
pgrep -f discord_bot_app > /dev/null && echo "✅ Bot running" || echo "❌ Bot not running"

# Check database
[ -f ./data/bot_dev.db ] && echo "✅ Database exists" || echo "❌ Database missing"

# Check memory
MEM=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
[ $MEM -lt 80 ] && echo "✅ Memory OK (${MEM}%)" || echo "⚠️ High memory (${MEM}%)"

# Check disk
DISK=$(df . | awk 'NR==2 {print $5}' | sed 's/%//')
[ $DISK -lt 90 ] && echo "✅ Disk OK (${DISK}%)" || echo "⚠️ Low disk space (${DISK}%)"
```

## 🆘 Getting Help

### Before Asking for Help
1. Run `python scripts/check_env.py` and share output
2. Share relevant logs (`logs/bot.log`)
3. Include your environment (OS, Python version)
4. Describe what you expected vs what happened

### Useful Commands for Support
```bash
# System info
python --version
pip list | grep discord
uname -a

# Configuration check
grep -E "(DISCORD_|STRIPE_|FEATURE_)" .env.dev

# Recent errors
grep -i error logs/bot.log | tail -10

# Database status
sqlite3 ./data/bot_dev.db ".tables"
sqlite3 ./data/bot_dev.db "SELECT COUNT(*) FROM users;"
```

### Community Resources
- Discord: [Join our server](https://discord.gg/your-invite)
- GitHub: [Open an issue](https://github.com/YOUR_USERNAME/discord-bot-starter-kit/issues)
- Documentation: Check `docs/` directory

## 📋 Emergency Recovery

### Bot Completely Down
```bash
# 1. Quick restart
systemctl restart discord-bot

# 2. Check why it failed
journalctl -u discord-bot -n 50

# 3. Reset to safe state
cp .env.example .env.dev
# Add only DISCORD_BOT_TOKEN_DEV
python -m src.app.discord_bot_app
```

### Database Corrupted
```bash
# 1. Stop bot
systemctl stop discord-bot

# 2. Backup corrupted DB
cp ./data/bot.db ./data/bot.db.corrupted

# 3. Start fresh (data loss!)
rm ./data/bot.db
systemctl start discord-bot
```

### Lost Discord Bot Token
1. Go to Discord Developer Portal
2. Your Application → Bot → Reset Token
3. Update environment file
4. Restart bot

### Stripe Webhook Issues
```bash
# 1. Check recent events
stripe listen --forward-to localhost:8000/webhooks/stripe/webhook

# 2. Test manually
curl -X POST http://localhost:8000/webhooks/stripe/webhook \
  -H "Stripe-Signature: test" \
  -d '{"type": "test"}'

# 3. Check logs
tail -f logs/api.log | grep webhook
```

## 🎯 Prevention Tips

1. **Regular backups**: Daily database backups
2. **Monitor logs**: Set up log alerts
3. **Test updates**: Staging environment first
4. **Document changes**: Keep track of modifications
5. **Security**: Rotate tokens quarterly
6. **Capacity planning**: Monitor resource usage

---

**Still stuck?**  
Run the diagnostic script and share the output with our support team.
