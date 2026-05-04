# Stripe Setup Guide

Complete guide for configuring Stripe for Discord bot user premium payments and SaaS tenant billing.

## Overview

This product uses Stripe for two independent billing flows:

1. **Discord User Premium** - Individual Discord users subscribe via `/pricing` command
2. **SaaS Tenant Billing** - Dashboard users subscribe via Billing page

These flows use separate Stripe webhooks and database tables. They do not synchronize automatically.

## Prerequisites

- Stripe account (test mode for development, live mode for production)
- Publicly accessible HTTPS endpoint for webhooks (or Stripe CLI for local testing)
- Domain name (for production webhooks)

## Step 1: Create Stripe Account

1. Go to [stripe.com](https://stripe.com) and sign up
2. Complete account verification (email, business details)
3. Enable test mode by default for development

## Step 2: Configure Discord User Premium Payments

### 2.1 Create Product and Price

1. Go to Stripe Dashboard → **Products**
2. Click **Add product**
3. Fill in product details:
   - **Name**: Your bot name (e.g., "Premium Bot Access")
   - **Description**: Description of premium features
   - **Pricing model**: Subscription
   - **Price**: Set your monthly/yearly price
   - **Currency**: Select your currency
4. Click **Create product**
5. Copy the **Price ID** (starts with `price_`)

**Example Price ID:** `price_1AbCdEfGhIjKlMnOpQrStUvW`

### 2.2 Configure Webhook for Discord Bot

1. Go to Stripe Dashboard → **Developers** → **Webhooks**
2. Click **Add endpoint**
3. Configure endpoint:
   - **Endpoint URL**: `https://your-domain.com/webhooks/stripe/webhook`
   - **Events to send**: Select these events:
     - `checkout.session.completed`
     - `customer.subscription.updated`
     - `customer.subscription.deleted`
     - `invoice.payment_succeeded`
     - `invoice.payment_failed`
4. Click **Add endpoint**
5. Copy the **Webhook signing secret** (starts with `whsec_`)

**Example Webhook Secret:** `whsec_abc123def456ghi789jkl012mno345pq`

### 2.3 Get API Keys

1. Go to Stripe Dashboard → **Developers** → **API keys**
2. Copy the **Secret key** (test mode for development)
3. Ensure it starts with `sk_test_...` for test mode or `sk_live_...` for live mode

**Example Secret Key:** `sk_test_51AbCdEfGhIjKlMnOpQrStUvWxYzAbCdEfGhIjKlMnOpQrStUvWxYzAbCdEf`

### 2.4 Configure Environment Variables

Add to your `.env` file:

```env
FEATURE_PAYMENTS=true
STRIPE_SECRET_KEY_DEV=sk_test_...
STRIPE_PRICE_ID_DEV=price_...
STRIPE_WEBHOOK_SECRET_DEV=whsec_...
STRIPE_WEBHOOK_SECRET_DISCORD_DEV=whsec_...
APP_BASE_URL_DEV=https://your-domain.com
```

For production:

```env
STRIPE_SECRET_KEY_PROD=sk_live_...
STRIPE_PRICE_ID_PROD=price_...
STRIPE_WEBHOOK_SECRET_PROD=whsec_...
STRIPE_WEBHOOK_SECRET_DISCORD_PROD=whsec_...
APP_BASE_URL_PROD=https://your-domain.com
```

## Step 3: Configure SaaS Tenant Billing (Optional)

### 3.1 Create SaaS Pricing Tiers

Create separate products for each SaaS plan tier:

#### Starter Plan

1. Create product: "SaaS Starter Plan"
2. Create recurring price for Starter tier
3. Copy Price ID to `STRIPE_SAAS_PRICE_STARTER_MONTHLY_DEV`

#### Pro Plan

1. Create product: "SaaS Pro Plan"
2. Create recurring price for Pro tier
3. Copy Price ID to `STRIPE_SAAS_PRICE_PRO_MONTHLY_DEV`

#### Enterprise Plan

1. Create product: "SaaS Enterprise Plan"
2. Create recurring price for Enterprise tier
3. Copy Price ID to `STRIPE_SAAS_PRICE_ENTERPRISE_MONTHLY_DEV`

### 3.2 Configure SaaS Webhook

1. Go to Stripe Dashboard → **Developers** → **Webhooks**
2. Create new endpoint or add to existing:
   - **Endpoint URL**: `https://your-domain.com/webhook`
   - **Events to send**: Same as Discord webhook events
3. Copy the Webhook signing secret

### 3.3 Configure Environment Variables

```env
STRIPE_SAAS_PRICE_STARTER_MONTHLY_DEV=price_...
STRIPE_SAAS_PRICE_PRO_MONTHLY_DEV=price_...
STRIPE_SAAS_PRICE_ENTERPRISE_MONTHLY_DEV=price_...
STRIPE_WEBHOOK_SECRET_SAAS_DEV=whsec_...
```

**Note:** Plans without configured Price IDs are excluded from the dashboard.

## Step 4: Local Testing with Stripe CLI

For local development without a public endpoint, use Stripe CLI to forward webhooks.

### 4.1 Install Stripe CLI

```bash
# macOS
brew install stripe/stripe-cli/stripe

# Linux
curl -s https://packages.stripe.dev/api/security/gpg-key/stripe-cli-el7.asc | sudo gpg --dearmor -o /usr/share/keyrings/stripe-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/stripe-archive-keyring.gpg] https://packages.stripe.dev/stripe-cli-el7/" | sudo tee /etc/apt/sources.list.d/stripe.list
sudo apt-get install stripe

# Windows
# Download from https://stripe.com/docs/stripe-cli
```

### 4.2 Login to Stripe

```bash
stripe login
```

Follow the prompts to authenticate.

### 4.3 Forward Webhooks

```bash
stripe listen --forward-to localhost:8000/webhooks/stripe/webhook
```

This will display a webhook signing secret for testing. Use this secret in your `.env.dev`:

```env
STRIPE_WEBHOOK_SECRET_DISCORD_DEV=whsec_test_...
```

### 4.4 Trigger Test Events

```bash
# Trigger checkout completion
stripe trigger checkout.session.completed

# Trigger subscription update
stripe trigger customer.subscription.updated

# Trigger payment failure
stripe trigger invoice.payment_failed
```

## Step 5: Production Configuration

### 5.1 Switch to Live Mode

1. Go to Stripe Dashboard
2. Toggle to **Live mode** in the top-right
3. Repeat product/price creation steps in live mode
4. Create webhooks in live mode
5. Get live API keys (start with `sk_live_...`)

### 5.2 Configure Production Environment

```env
APP_ENV=prod
STRIPE_SECRET_KEY_PROD=sk_live_...
STRIPE_PRICE_ID_PROD=price_...
STRIPE_WEBHOOK_SECRET_PROD=whsec_...
STRIPE_WEBHOOK_SECRET_DISCORD_PROD=whsec_...
STRIPE_WEBHOOK_SECRET_SAAS_PROD=whsec_...
APP_BASE_URL_PROD=https://your-bot-domain.com
```

### 5.3 Verify Webhook Endpoint

Ensure your webhook endpoint is publicly accessible:

```bash
curl -X POST https://your-domain.com/webhooks/stripe/webhook \
  -H "Content-Type: application/json" \
  -d '{"type": "test"}'
```

Expected response: HTTP 200 or 400 (signature verification error is expected for invalid signatures).

## Step 6: Test Payment Flow

### Test Mode

1. Run `/pricing` in Discord
2. Select a plan
3. Use Stripe test card: `4242 4242 4242 4242`
4. Complete checkout
5. Verify `user_profile.is_premium` is set to 1 in database
6. Test `/premium` command access

### Live Mode

1. Use a real payment method
2. Complete checkout
3. Verify premium access is granted
4. Check Stripe Dashboard for transaction

## Webhook Event Handling

### checkout.session.completed

Triggered when user completes payment. Bot:
- Verifies webhook signature
- Extracts `discord_user_id` from metadata
- Sets `user_profile.is_premium = 1`
- Stores `stripe_subscription_id` and `stripe_customer_id`
- Logs event to `stripe_events` table for idempotency

### customer.subscription.updated

Triggered when subscription changes (upgrade/downgrade). Bot:
- Updates subscription metadata
- May adjust access level based on new plan

### customer.subscription.deleted

Triggered when subscription is cancelled. Bot:
- Sets `user_profile.is_premium = 0`
- Clears subscription IDs
- Removes premium access

### invoice.payment_succeeded

Triggered when payment succeeds. Bot:
- Logs successful payment
- May extend premium access

### invoice.payment_failed

Triggered when payment fails. Bot:
- Logs failed payment
- May send notification to user
- May revoke premium access after retry period

## Idempotency

Webhook processing is idempotent via the `stripe_events` table:

1. Each webhook event has a unique `event_id`
2. Bot checks if event was already processed
3. If processed, skips processing (prevents duplicate premium activation)
4. If not processed, processes and marks as completed

**Query to check processed events:**

```sql
SELECT event_id, type, status, created_at 
FROM stripe_events 
ORDER BY created_at DESC 
LIMIT 10;
```

## Troubleshooting

### Webhook Not Receiving Events

**Symptoms:** Payments complete but premium not activated

**Solutions:**
1. Verify webhook URL is correct in Stripe Dashboard
2. Check webhook URL is publicly accessible (use curl)
3. Verify webhook secret matches exactly
4. Check server logs for webhook errors
5. Use Stripe CLI for local testing

### Signature Verification Failed

**Symptoms:** `stripe.error.SignatureVerificationError`

**Solutions:**
1. Verify webhook secret matches exactly (no extra spaces)
2. Check request headers include `Stripe-Signature`
3. Ensure request body is not modified
4. Check for encoding issues

### Premium Not Activated After Payment

**Solutions:**
1. Check `stripe_events` table for processed events
2. Verify `discord_user_id` is in checkout session metadata
3. Check `user_profile` table for premium status
4. Manually trigger webhook from Stripe Dashboard
5. Check for database errors in logs

### Test vs Live Keys Mixed

**Error:** "Using test key in prod environment"

**Solutions:**
1. Dev: Use `sk_test_...` keys
2. Prod: Use `sk_live_...` keys
3. Never mix test and live keys
4. Application enforces this validation

### Webhook Endpoint Not Accessible

**Solutions:**
1. Ensure nginx is running and configured
2. Check firewall allows port 443
3. Verify SSL certificate is valid
4. Test with curl from external network
5. Use ngrok for testing: `ngrok http 8000`

## Best Practices

1. **Test thoroughly in test mode** before using live mode
2. **Use Stripe CLI** for local webhook testing
3. **Monitor webhook failures** in Stripe Dashboard
4. **Set up alerts** for failed payments
5. **Implement retry logic** for failed webhooks
6. **Keep webhook secrets secure** - never commit to git
7. **Use separate webhooks** for Discord and SaaS flows
8. **Log all webhook events** for debugging
9. **Regularly reconcile** Stripe and database records
10. **Implement customer support** for payment issues

## Security Considerations

1. **Always verify webhook signatures** - never disable
2. **Use HTTPS** for all webhook endpoints in production
3. **Rotate webhook secrets** periodically
4. **Never log raw request bodies** (may contain card data)
5. **Validate all data** from webhooks before database writes
6. **Use Stripe's test cards** only in test mode
7. **Implement rate limiting** on webhook endpoints
8. **Monitor for suspicious activity** in Stripe Dashboard

## Stripe Test Cards

Use these cards in test mode:

| Card Number | Description |
|-------------|-------------|
| `4242 4242 4242 4242` | Success |
| `4000 0025 0000 3155` | Requires 3D Secure |
| `4000 0000 0000 0002` | Card declined |
| `4000 0000 0000 9995` | Insufficient funds |
| `4000 0000 0000 0069` | Expired card |

**Expiration:** Any future date  
**CVC:** Any 3 digits  
**Postal Code:** Any 5 digits

## Additional Resources

- [Stripe Documentation](https://stripe.com/docs)
- [Stripe Webhooks Guide](https://stripe.com/docs/webhooks)
- [Stripe CLI Documentation](https://stripe.com/docs/stripe-cli)
- [Stripe Test Cards](https://stripe.com/docs/testing)

## Billing Troubleshooting

For specific billing issues, see [Billing Troubleshooting Guide](BILLING_TROUBLESHOOTING.md).
