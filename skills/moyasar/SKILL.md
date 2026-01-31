---
name: moyasar
description: Integrate Moyasar payment gateway for Saudi Arabia. Covers web (hosted form, custom UI), iOS, Android, Flutter, React Native. Supports cards, Mada, Apple Pay, Samsung Pay, STC Pay. Use when building checkout, payment processing, or payment links.
---

# Moyasar Payments

Saudi Arabia's primary payment gateway.

## Core Rules

**Amounts**: Always in halalas (1 SAR = 100 halalas). Minimum: 100.

**API Keys**:
| Key | Prefix | Use |
|-----|--------|-----|
| Publishable | `pk_test_` / `pk_live_` | Client-side only |
| Secret | `sk_test_` / `sk_live_` | Server-side only |

**Payment Statuses**: `initiated` → `paid` | `failed` | `authorized` → `captured` → `refunded` | `voided`

## Handling Failed Payments

When `status === 'failed'`, check `source.message` and `source.response_code`:

| Code | Reason |
|------|--------|
| `14` | Invalid Card Number |
| `51` | Insufficient Funds |
| `54` | Expired Card |
| `55` | Incorrect PIN |
| `82` | CVV Validation Error |
| `96` | System Error |

See **[errors.md](references/errors.md)** for full error list with English/Arabic translations.

## The Trust Problem

**Never trust the client.** After payment, Moyasar redirects to your `callback_url` with status. But users can manipulate URLs, close browsers, or have network issues.

### Verification Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  CLIENT                           SERVER                        │
│                                                                 │
│  1. User pays via Moyasar ───────────────────────────────────►  │
│                                                                 │
│  2. Redirect to callback_url?id=xxx&status=paid                 │
│     → Show success/failure message (OK for UX)                  │
│     → But do NOT grant access or confirm purchase yet           │
│                                                                 │
│                                   3. VERIFY before fulfilling:  │
│                                      Option A: Fetch payment    │
│                                      Option B: Webhook          │
│                                                                 │
│  4. Grant access / confirm  ◄────────────────────────────────   │
│     only after server verifies                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Option A - Fetch Payment** (simpler, synchronous):
```javascript
// On callback, verify with Moyasar API
const payment = await fetch(`https://api.moyasar.com/v1/payments/${id}`, {
  headers: { 'Authorization': `Basic ${btoa(process.env.MOYASAR_SECRET_KEY + ':')}` }
}).then(r => r.json());

if (payment.status === 'paid' && payment.amount === expectedAmount) {
  // Safe to fulfill
}
```

**Option B - Webhooks** (recommended for reliability):
- Moyasar POSTs to your server when payment status changes
- Retries on failure (1min → 10min → 30min → 1hr → 2hr)
- See [webhooks.md](references/webhooks.md)

## Platform Guide

### Web Applications

See **[web.md](references/web.md)** for complete guide.

**Two approaches:**

| Approach | Use When |
|----------|----------|
| **Hosted Form** | Quick integration, PCI compliance handled |
| **Custom UI** | Need full design control, willing to handle tokenization |

**Quick start (Hosted Form):**
```html
<script src="https://cdn.moyasar.com/mpf/1.14.0/moyasar.js"></script>
<link rel="stylesheet" href="https://cdn.moyasar.com/mpf/1.14.0/moyasar.css">
<div class="mysr-form"></div>
<script>
Moyasar.init({
  element: '.mysr-form',
  amount: 10000,
  currency: 'SAR',
  description: 'Order #123',
  publishable_api_key: process.env.MOYASAR_PUBLISHABLE_KEY,
  callback_url: 'https://yoursite.com/callback',
  methods: ['creditcard', 'applepay', 'stcpay'],
  metadata: { order_id: '123' }  // Link payment to your order
});
</script>
```

### Mobile Applications

| Platform | Reference | Package |
|----------|-----------|---------|
| iOS | [ios.md](references/ios.md) | `MoyasarSdk` (SPM/CocoaPods) |
| Android | [android.md](references/android.md) | `com.github.Moyasar:moyasar-android-sdk` |
| Flutter | [flutter.md](references/flutter.md) | `moyasar` (pub.dev) |
| React Native | [react-native.md](references/react-native.md) | `react-native-moyasar-sdk` |

### Server-Side Operations

See **[api.md](references/api.md)** for:
- Verify payments
- Refunds (full/partial)
- Capture/void (pre-auth)
- Tokenization (save cards)
- Invoices (payment links)

## Reference Files

| File | When to Read |
|------|--------------|
| [web.md](references/web.md) | Building web checkout (hosted form, custom UI, Apple Pay, STC Pay) |
| [ios.md](references/ios.md) | iOS app integration |
| [android.md](references/android.md) | Android app integration |
| [flutter.md](references/flutter.md) | Flutter app integration |
| [react-native.md](references/react-native.md) | React Native app integration |
| [api.md](references/api.md) | Server-side operations (verify, refund, capture, tokens, invoices) |
| [webhooks.md](references/webhooks.md) | Setting up payment webhooks |
| [errors.md](references/errors.md) | Error codes with English/Arabic translations |
| [test-data.md](references/test-data.md) | Test cards, STC Pay Testing, Apple Pay Testing |

## Key Gotchas

1. **Always use metadata** - Include `order_id` to link payment back to your system
2. **Webhook secret is in BODY, not header** - Moyasar sends `secret_token` in request body
3. **Idempotency** - Use `given_id` (UUID) when creating payment to prevent duplicate charges on retry
4. **Apple Pay needs domain verification** - Host file at `/.well-known/apple-developer-merchantid-domain-association`
