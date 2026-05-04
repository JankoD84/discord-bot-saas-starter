# Discord App Setup Guide

Complete guide for configuring a Discord application, bot, and OAuth integration.

## Overview

This guide covers:
1. Creating a Discord application
2. Configuring the bot with required intents
3. Setting up OAuth2 for bot invitations
4. Configuring Discord OAuth for SaaS integration
5. Required permissions and scopes

## Prerequisites

- Discord account
- Discord Developer Portal access

## Step 1: Create Discord Application

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **New Application**
3. Enter application name (e.g., "My Bot")
4. Click **Create**

## Step 2: Configure Bot

### 2.1 Create Bot User

1. In your application, navigate to **Bot** in the left sidebar
2. Click **Add Bot**
3. Confirm by clicking **Yes, do it!**

### 2.2 Enable Privileged Gateway Intents

Under **Privileged Gateway Intents**, enable:

- **Message Content Intent** - Required for reading message content
- **Server Members Intent** - Required for member-related features
- **Presence Intent** - Optional, for presence features

**Important:** These intents are required for the bot to function properly. Without them, the bot will fail to start or certain features will not work.

### 2.3 Copy Bot Token

1. Under **Token**, click **Reset Token** (if first setup) or **Copy**
2. **Important:** This token is only shown once. Copy it immediately.
3. Store this token in your environment variables:

```env
DISCORD_BOT_TOKEN_DEV=your_bot_token_here
```

**Security:** Never commit bot tokens to version control. Reset the token immediately if exposed.

## Step 3: Configure OAuth2 for Bot Invitation

### 3.1 Generate OAuth2 URL

1. Navigate to **OAuth2** → **URL Generator**
2. Under **Scopes**, select:
   - `bot` - Required for bot functionality
   - `applications.commands` - Required for slash commands

### 3.2 Configure Bot Permissions

Under **Bot Permissions**, select the required permissions:

**Essential Permissions:**
- **Send Messages** - Required for bot responses
- **Use Slash Commands** - Required for slash command interaction
- **Embed Links** - Required for rich embeds
- **Read Message History** - Required for context

**Optional Permissions (if needed):**
- **Manage Messages** - For message management features
- **Kick Members** - For moderation features
- **Ban Members** - For moderation features
- **Add Reactions** - For reaction features
- **Attach Files** - For file uploads

### 3.3 Copy Invite URL

1. Scroll to the bottom
2. Copy the generated URL
3. Paste in browser to invite bot to your server
4. Select server and authorize

## Step 4: Configure Discord OAuth for SaaS Integration (Optional)

This section is only required if using Discord OAuth for SaaS dashboard authentication.

### 4.1 Enable OAuth2

1. Navigate to **OAuth2** → **General**
2. Under **Default Authorization Link**, note the Client ID

### 4.2 Add Redirect URIs

1. Under **Redirects**, click **Add Redirect**
2. Add your frontend URL (not backend callback):
   - Development: `http://localhost:8000/`
   - Production: `https://your-bot-domain.com/`

**Important:** The redirect URI should point to your frontend SPA, not the backend callback endpoint. The frontend will extract the code and call the backend.

### 4.3 Copy Client Secret

1. Under **Client Secret**, click **Reset Secret** (if first setup) or **Copy**
2. Store in environment variables:

```env
DISCORD_CLIENT_ID_DEV=your_client_id_here
DISCORD_CLIENT_SECRET_DEV=your_client_secret_here
DISCORD_REDIRECT_URI_DEV=http://localhost:8000/
```

For production:

```env
DISCORD_CLIENT_ID_PROD=your_client_id_here
DISCORD_CLIENT_SECRET_PROD=your_client_secret_here
DISCORD_REDIRECT_URI_PROD=https://your-bot-domain.com/
```

## Step 5: Get Server ID (Optional but Recommended)

Having your server ID speeds up command registration in development.

### 5.1 Enable Developer Mode

1. Go to Discord (desktop or web)
2. User Settings → Advanced
3. Enable **Developer Mode**

### 5.2 Copy Server ID

1. Right-click your server name
2. Select **Copy Server ID**
3. Store in environment variables:

```env
DISCORD_GUILD_ID_DEV=your_server_id_here
```

**Benefit:** With guild ID set, slash commands sync in 1-5 seconds. Without it, global sync can take up to 1 hour.

## Step 6: Configure Environment Variables

Complete your `.env` file with Discord configuration:

```env
APP_ENV=dev

# Discord Bot Configuration
DISCORD_BOT_TOKEN_DEV=your_bot_token_here
DISCORD_GUILD_ID_DEV=your_server_id_here

# Discord OAuth (only if using SaaS OAuth)
DISCORD_CLIENT_ID_DEV=your_client_id_here
DISCORD_CLIENT_SECRET_DEV=your_client_secret_here
DISCORD_REDIRECT_URI_DEV=http://localhost:8000/
```

## Step 7: Verify Bot Setup

### 7.1 Start the Bot

```bash
python -m src.app.discord_bot_app
```

Expected logs:
```
✅ DISCORD BOT CONNECTED SUCCESSFULLY
🤖 Bot Username: YourBotName
🆔 Bot ID: 123456789012345678
📋 Registered X slash commands:
   • /ping
   • /demo
   • /pricing
   • /status
🎉 BOT IS READY AND LISTENING FOR COMMANDS
```

### 7.2 Test Commands

In your Discord server:

```
/ping
```

Expected: Bot responds with status embed.

```
/status
```

Expected: Bot responds with system health overview.

## Troubleshooting

### Bot Won't Connect

**Error:** `PrivilegedIntentsRequired`

**Solution:** Ensure Message Content Intent and Server Members Intent are enabled in Discord Developer Portal.

**Error:** `Invalid token`

**Solution:** 
- Verify token is 50+ characters
- Check for extra spaces or line breaks
- Get a fresh token from Discord Developer Portal

### Commands Not Appearing

**Symptoms:** Slash commands don't show in Discord

**Solutions:**
1. Wait 1-5 minutes for global sync (or set DISCORD_GUILD_ID for instant sync)
2. Reinvite the bot using fresh invite URL
3. Check bot has `applications.commands` scope
4. Verify bot is running and connected

### "Interaction Failed" Error

**Symptoms:** Commands return "Interaction failed"

**Solutions:**
1. Bot restarted while command was executing
2. Missing permissions (check bot has Send Messages)
3. Network timeout
4. Try command again

### OAuth Callback Fails

**Symptoms:** Discord OAuth redirect fails

**Solutions:**
1. Verify redirect URI matches exactly (including trailing slash)
2. Ensure redirect URI is added in Discord Developer Portal
3. Check frontend is extracting code correctly
4. Verify backend callback endpoint is accessible

## Security Best Practices

1. **Never commit bot tokens** to version control
2. **Reset tokens immediately** if exposed
3. **Use separate applications** for dev and prod
4. **Limit bot permissions** to minimum required
5. **Enable two-factor authentication** on Discord account
6. **Regularly audit bot permissions** and remove unused ones
7. **Monitor bot's action log** for suspicious activity
8. **Use rate limiting** to prevent API abuse

## Bot Permissions Reference

### Permission Integer Calculator

Use [Discord Permission Calculator](https://discordapi.com/permissions.html) to calculate permission integers.

### Common Permission Sets

**Minimal Bot Permissions:**
- Send Messages (2048)
- Use Slash Commands (2147483648)
- Embed Links (16384)
- Read Message History (65536)
- **Total:** 2148009792

**Moderation Bot Permissions:**
- Minimal Bot Permissions (2148009792)
- Manage Messages (8192)
- Kick Members (2)
- Ban Members (4)
- **Total:** 2148017990

## OAuth2 Scopes Reference

### Required Scopes

| Scope | Purpose |
|-------|---------|
| `bot` | Bot functionality |
| `applications.commands` | Slash commands |

### Optional Scopes

| Scope | Purpose |
|-------|---------|
| `identify` | User identity (OAuth) |
| `email` | User email (OAuth) |
| `guilds` | Server access (OAuth) |
| `guilds.join` | Join servers (OAuth) |

## Production Considerations

### Separate Applications

Use separate Discord applications for:
- Development environment
- Production environment

This prevents accidental production changes during development.

### Bot Verification

For bots in 100+ servers, Discord requires bot verification:
- Complete bot verification process in Discord Developer Portal
- Provide bot description and terms of service
- Ensure bot follows Discord's Terms of Service

### Rate Limits

Discord API has rate limits:
- Global rate limit: 50 requests per second
- Per-channel rate limit: 5 requests per second
- The bot handles rate limiting automatically via discord.py

### Bot Status

Set a custom status in `discord_bot_app.py`:

```python
await bot.change_presence(activity=discord.Activity(
    type=discord.ActivityType.watching,
    name="for /help commands"
))
```

## Additional Resources

- [Discord Developer Portal](https://discord.com/developers/applications)
- [Discord.py Documentation](https://discordpy.readthedocs.io/)
- [Discord API Documentation](https://discord.com/developers/docs/intro)
- [Discord OAuth2 Documentation](https://discord.com/developers/docs/topics/oauth2)

## Verification Checklist

Before going to production:

- [ ] Bot token stored securely in environment variables
- [ ] Required intents enabled (Message Content, Server Members)
- [ ] OAuth2 URL generated with correct scopes and permissions
- [ ] Bot invited to production server with correct permissions
- [ ] Discord Guild ID configured for faster command sync
- [ ] Discord OAuth configured (if using SaaS integration)
- [ ] Redirect URIs added for OAuth (if using SaaS integration)
- [ ] Bot tested in production environment
- [ ] Bot verification completed (if in 100+ servers)
- [ ] Two-factor authentication enabled on Discord account
