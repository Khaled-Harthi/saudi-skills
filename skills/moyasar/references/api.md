# Server-Side API

Base URL: `https://api.moyasar.com/v1`

## Authentication

Basic Auth with secret key as username, **empty password**.

```javascript
const headers = {
  'Authorization': `Basic ${Buffer.from(process.env.MOYASAR_SECRET_KEY + ':').toString('base64')}`,
  'Content-Type': 'application/json'
};
```

---

## Endpoints

### Payments

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/payments/{id}` | Verify payment status |
| `POST` | `/payments` | Create payment |
| `POST` | `/payments/{id}/refund` | Refund (full or partial) |
| `POST` | `/payments/{id}/capture` | Capture pre-auth |
| `POST` | `/payments/{id}/void` | Void pre-auth |

### Tokens

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/tokens/{id}` | Get token info |
| `DELETE` | `/tokens/{id}` | Delete saved card |

### Invoices

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/invoices` | Create payment link |

---

## Create Payment

```
POST /payments
```

| Field | Required | Description |
|-------|----------|-------------|
| `amount` | Yes | Amount in halalas |
| `currency` | Yes | `SAR`, `USD`, etc. |
| `source` | Yes | Payment source (see below) |
| `callback_url` | Yes | Redirect URL after payment |
| `given_id` | No | UUID for idempotency |
| `description` | No | Order description |
| `metadata` | No | Custom key-value pairs |

**Source Types:**

| Type | Fields |
|------|--------|
| `token` | `{ type: 'token', token: 'tok_...' }` |
| `creditcard` | `{ type: 'creditcard', name, number, month, year, cvc }` |
| `stcpay` | `{ type: 'stcpay', mobile: '05XXXXXXXX' }` |
| `applepay` | `{ type: 'applepay', token: '...' }` |

**Response:** Payment object. If `status === 'initiated'`, redirect to `source.transaction_url` for 3DS.

---

## Verify Payment

```
GET /payments/{id}
```

**Response:**
```json
{
  "id": "pay_...",
  "status": "paid",
  "amount": 10000,
  "currency": "SAR",
  "metadata": { "order_id": "123" }
}
```

Always verify `status === 'paid'` AND `amount === expectedAmount` before fulfilling.

---

## Refund

```
POST /payments/{id}/refund
```

| Field | Required | Description |
|-------|----------|-------------|
| `amount` | No | Partial refund amount (omit for full) |

---

## Capture & Void

For pre-authorized payments (created with `manual: true`).

```
POST /payments/{id}/capture
POST /payments/{id}/void
```

Issuer auto-voids if not captured within ~7 days.

---

## Tokens

Save card during payment with `save_card: true`. Then:

```
GET /tokens/{id}
```
Returns: `brand`, `last4`, `name`, `expires`

```
DELETE /tokens/{id}
```
Returns: 204 No Content

---

## Invoices

Shareable payment links without building UI.

```
POST /invoices
```

| Field | Required | Description |
|-------|----------|-------------|
| `amount` | Yes | Amount in halalas |
| `currency` | Yes | Currency code |
| `description` | Yes | Invoice description |
| `callback_url` | No | Webhook URL |
| `success_url` | No | Redirect after payment |
| `back_url` | No | Cancel/back URL |
| `expired_at` | No | ISO 8601 expiry date |

**Response:** Invoice object with `url` to share with customer.

---

## Notes

**Metadata:** 30 keys max, 40 char key names, 500 char values.

**Idempotency:** Use `given_id` (UUID) to prevent duplicate charges on retry.
