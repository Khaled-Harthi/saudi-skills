# Webhooks

Moyasar POSTs to your server when payment status changes.

## Setup

Dashboard → Settings → Webhooks → Add webhook

- **Endpoint**: HTTPS URL
- **Secret Token**: For verification
- **Events**: Which events to receive

## Verification

**Critical:** `secret_token` is in request BODY, not header.

```javascript
app.post('/webhooks/moyasar', (req, res) => {
  if (req.body.secret_token !== process.env.MOYASAR_WEBHOOK_SECRET) {
    return res.status(401).send('Unauthorized');
  }

  const { type, data } = req.body;

  // Use data.id to prevent duplicate processing
  if (await alreadyProcessed(data.id)) {
    return res.status(200).send('OK');
  }

  // metadata is set by you when creating payment (null for invoice payments)
  const orderId = data.metadata?.order_id;

  switch (type) {
    case 'payment_paid':
      await fulfillOrder(orderId, data);
      break;
    case 'payment_failed':
      await notifyFailure(orderId, data.source?.message);
      break;
  }

  res.status(200).send('OK');  // Respond quickly
});
```

## Events

| Event | When |
|-------|------|
| `payment_paid` | Success |
| `payment_failed` | Declined |
| `payment_authorized` | Pre-auth approved |
| `payment_captured` | Auth captured |
| `payment_refunded` | Refund processed |
| `payment_voided` | Cancelled |

## Payload Example

```json
{
  "id": "bf8fee8c-8bc0-4948-bde5-b8d06f97df00",
  "type": "payment_paid",
  "secret_token": "your_webhook_secret",
  "live": true,
  "created_at": "2026-01-31T09:05:03+00:00",
  "data": {
    "id": "0f3d87c2-88ae-438a-82ec-51cc42a1e1a7",
    "status": "paid",
    "amount": 29000,
    "fee": 449,
    "currency": "SAR",
    "description": "Test Purchase",
    "metadata": {
      "order_id": "123",
      "userId": "0f3d87c2-8bc0-4948-bde5-51cc42a1e1a7"
    },
    "invoice_id": null,
    "source": {
      "type": "creditcard",
      "company": "mada",
      "name": "Ahmed Mohammed",
      "number": "4200-23XX-XXXX-4111",
      "message": "APPROVED",
      "response_code": "00",
      "issuer_name": "Al Rajhi Banking & Inv. Corp.",
      "issuer_country": "SA",
      "reference_number": "600000000000"
    },
    "created_at": "2026-01-31T09:04:44.791Z",
    "amount_format": "290.00 SAR",
    "fee_format": "4.49 SAR"
  }
}
```

## Key Fields

| Field | Description |
|-------|-------------|
| `live` | `true` = production payment, `false` = test payment (Moyasar doesn't use separate webhook endpoints for test vs production — the live flag is how you distinguish them.) |
| `data.id` | Payment ID (use for deduplication) |
| `data.amount` | Amount in halalas |
| `data.fee` | Moyasar fee in halalas |
| `data.metadata` | Your custom data |
| `data.source.type` | `creditcard`, `applepay`, `stcpay`, `samsungpay` |
| `data.source.company` | `mada`, `visa`, `mastercard`, `amex` |

## Source Types

**Credit Card / Apple Pay / Samsung Pay:**
- `company`, `number`, `name`
- `issuer_name`, `issuer_country`, `issuer_card_type`
- `response_code`, `reference_number`, `authorization_code`

**STC Pay:**
- `mobile`, `reference_number`
- `branch`, `cashier`

## Retry Schedule

Failed webhooks (non-2xx) retry: 1min → 10min → 30min → 1hr → 2hr → dropped

## Best Practices

1. **Respond quickly** - Return 2xx within seconds, process async if needed
2. **Deduplicate** - Use `data.id` to prevent double-processing
3. **Use metadata** - Include `order_id` when creating payment to link back to your system