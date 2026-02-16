---
name: authentica-otp
description: Integrate Authentica OTP verification for Saudi Arabia. Send and verify OTPs via SMS. Use when building phone verification, two-factor authentication, or OTP-based login flows.
---

# Authentica OTP Verification

Saudi OTP verification service via SMS.

See **[setup.md](references/setup.md)** for account creation, API keys, credits, and sender ID registration.

## Core Rules

**Base URL**: `https://api.authentica.sa/api/v2`

**Authentication**: API key in `X-Authorization` header.

**Phone Format**: International E.164 only — `+9665XXXXXXXX`, never `05XXXXXXXX`.

**Templates**: Selected by `template_id`. Default is `1`. Authentica provides pre-built templates in Arabic and English — browse available templates and their IDs in the Authentica portal under **Templates**. Users can request new templates but they are subject to approval.

**Response difference**: `send-otp` returns `success` (boolean). `verify-otp` returns `status` (boolean). They are not the same field.

## Send OTP

```
POST /send-otp
```

| Field | Required | Description |
|-------|----------|-------------|
| `method` | Yes | `sms` |
| `phone` | Yes | International format: `+9665XXXXXXXX` |
| `template_id` | No | Message template ID. Default `1`. |
| `otp` | No | Custom OTP (numbers only). Omit to let Authentica generate one. |

```bash
curl -X POST 'https://api.authentica.sa/api/v2/send-otp' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Authorization: YOUR_API_KEY' \
  -d '{
    "method": "sms",
    "phone": "+9665XXXXXXXX",
    "template_id": 1
  }'
```

**Success**: `{ "success": true, "message": "..." }`
**Failure**: `{ "success": false, "message": "..." }`

## Verify OTP

```
POST /verify-otp
```

| Field | Required | Description |
|-------|----------|-------------|
| `phone` | Yes | Same phone number the OTP was sent to. |
| `otp` | Yes | OTP code entered by the user. |

```bash
curl -X POST 'https://api.authentica.sa/api/v2/verify-otp' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Authorization: YOUR_API_KEY' \
  -d '{
    "phone": "+9665XXXXXXXX",
    "otp": "123456"
  }'
```

**Success**: `{ "status": true, "message": "..." }`
**Failure**: `{ "status": false, "message": "..." }`

## Environment Variables

| Variable | Description |
|----------|-------------|
| `AUTHENTICA_API_KEY` | API key from [Authentica dashboard](https://portal.authentica.sa/settings/apikeys/) |

## Custom Sender ID

Custom sender ID needs legal docs — Requires commercial registration (سجل تجاري) or intellectual property certificate (شهادة ملكية فكرية) from SAIP (Saudi Authority for Intellectual Property / الهيئة السعودية للملكية الفكرية). Use default sender while waiting. See [setup.md](references/setup.md).
