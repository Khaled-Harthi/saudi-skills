# Web Integration

## Contents
- [Hosted Form](#hosted-form) - Quick integration
- [Custom UI](#custom-ui) - Full design control
- [Apple Pay](#apple-pay) - Requires domain verification
- [STC Pay](#stc-pay) - Saudi mobile wallet
- [Samsung Pay](#samsung-pay) - Requires Samsung developer account

---

## Hosted Form

Moyasar handles UI, 3DS, and PCI compliance.

```html
<script src="https://cdn.moyasar.com/mpf/1.14.0/moyasar.js"></script>
<link rel="stylesheet" href="https://cdn.moyasar.com/mpf/1.14.0/moyasar.css">
<div class="mysr-form"></div>
```

```javascript
Moyasar.init({
  element: '.mysr-form',
  amount: 10000,  // halalas
  currency: 'SAR',
  description: 'Order #123',
  publishable_api_key: 'pk_test_...', // MOYASAR_PUBLISHABLE_KEY
  callback_url: 'https://yoursite.com/callback',
  methods: ['creditcard', 'applepay', 'stcpay'],
  supported_networks: ['mada', 'visa', 'mastercard'],
  metadata: { order_id: '123' }
});
```

After payment, user redirects to `callback_url?id=xxx&status=paid&message=...`

**Always verify server-side** (see SKILL.md).

---

## Custom UI

For full design control. You handle the form, Moyasar handles PCI-compliant tokenization.

### Step 1: Tokenize (Client-Side)

Collect card details in your form, then tokenize with your **publishable key**. Must use `multipart/form-data`:

```javascript
const form = new FormData();
form.append('publishable_api_key', 'pk_test_...');
form.append('name', 'John Doe');
form.append('number', '4111111111111111');
form.append('month', '09');
form.append('year', '2025');
form.append('cvc', '123');
form.append('callback_url', 'https://yoursite.com/callback');

const res = await fetch('https://api.moyasar.com/v1/tokens', {
  method: 'POST',
  body: form  // Not JSON
});
const { id: tokenId } = await res.json();  // "tok_..."
```

### Step 2: Create Payment (Server-Side)

Send `tokenId` to your server. Create payment with your **secret key**:

```javascript
const payment = await fetch('https://api.moyasar.com/v1/payments', {
  method: 'POST',
  headers: {
    'Authorization': `Basic ${btoa(secretKey + ':')}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    amount: 10000,  // From YOUR database, not client
    currency: 'SAR',
    source: { type: 'token', token: tokenId },
    callback_url: 'https://yoursite.com/callback'
  })
});

// If 3DS required, redirect user
if (payment.status === 'initiated') {
  window.location.href = payment.source.transaction_url;
}
```

---

## Apple Pay

### Prerequisites
1. HTTPS required
2. Safari/iOS only
3. Domain verified with Moyasar

### Domain Verification
1. **Moyasar Dashboard** → Settings → Apple Pay Domains
2. Download verification file
3. Host at: `https://yoursite.com/.well-known/apple-developer-merchantid-domain-association`
4. Click Validate, then Register

### Configuration

```javascript
Moyasar.init({
  // ... base config
  methods: ['applepay'],
  apple_pay: {
    country: 'SA',
    label: 'Your Store',
    validate_merchant_url: 'https://api.moyasar.com/v1/applepay/initiate'
  }
});
```

---

## STC Pay

Saudi mobile wallet with OTP verification.

### Prerequisites
1. Merchant ID from STC Pay portal
2. Activation from Moyasar account manager

### Configuration

```javascript
Moyasar.init({
  // ... base config
  methods: ['stcpay']
});
```

Flow: User enters mobile (05XXXXXXXX) → OTP → Returns to `callback_url`

---

## Samsung Pay

### Prerequisites
1. Samsung Developer Account + Service ID
2. Domain registered with Samsung
3. CSR uploaded in Moyasar Dashboard → Settings → Samsung Certificate

### Configuration

```javascript
Moyasar.init({
  // ... base config
  methods: ['samsungpay'],
  samsung_pay: {
    service_id: 'your_service_id',
    order_number: 'ORDER_123',
    country: 'SA',
    environment: 'PRODUCTION'  // or 'STAGE'
  }
});
```

---

## Configuration Reference

### Required

| Parameter | Description |
|-----------|-------------|
| `element` | CSS selector |
| `amount` | Amount in halalas |
| `currency` | `'SAR'`, `'USD'`, etc. |
| `publishable_api_key` | Your publishable key |
| `callback_url` | Redirect after payment |

### Optional

| Parameter | Default | Description |
|-----------|---------|-------------|
| `methods` | All | `['creditcard', 'applepay', 'stcpay', 'samsungpay']` |
| `supported_networks` | All except amex | `['mada', 'visa', 'mastercard', 'amex']` |
| `language` | From HTML | `'en'`, `'ar'` |
| `metadata` | — | Custom key-value pairs |
| `credit_card.save_card` | false | Tokenize for future use |
| `credit_card.manual` | false | Authorize only, capture later |

### Callbacks

| Callback | Description |
|----------|-------------|
| `on_initiating` | Before payment. Return `false` to cancel |
| `on_completed` | Payment created, before 3DS redirect. **Save payment ID here** |
| `on_failure` | Network/validation error |
| `on_redirect` | Intercept redirect (for SPAs) |

### Save Payment ID (Recommended)

Use `on_completed` to save the payment ID to your backend **before** the 3DS redirect. This enables a simpler verification flow:

1. `on_completed` fires → save payment ID + order info to your database
2. User redirects back via `callback_url` → trust the client status for UX (show success/error)
3. Webhook arrives later → match payment ID, verify status, then action it (enable subscription, confirm purchase, etc.)

This avoids needing an immediate server-side API call to verify every payment. It also protects against connection drops during redirect — you already have the payment record.

```javascript
Moyasar.init({
  // ... base config
  on_completed: async function (payment) {
    await savePaymentOnBackend(payment);
  }
});
```
