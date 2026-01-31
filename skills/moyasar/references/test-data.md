# Test Data

Use test API keys (`pk_test_`, `sk_test_`) with this data.

## Card Numbers

### Success

| Card | Brand |
|------|-------|
| `4111111111111111` | Visa |
| `4111114005765430` | Visa (frictionless 3DS) |
| `5421080101000000` | Mastercard |
| `340000000900000` | Amex |
| `4201320111111010` | Mada |

### 3DS Scenarios

| Card | Behavior |
|------|----------|
| `4000000000003220` | 3DS challenge required |
| `4111112205628150` | 3DS auth fails |
| `4111116611600661` | Not 3DS enrolled |

### Decline

| Card | Error |
|------|-------|
| `4201320000013020` | Unspecified failure |
| `4201320000311101` | Insufficient funds |
| `4201320131000508` | Lost card |
| `4201321234411220` | Declined |
| `4201322267774310` | Expired card |
| `4201326324640570` | Exceeds limit |
| `4201321144311528` | Stolen card |

### Card Details

- **Name**: Any (2+ words)
- **Expiry**: Any future date
- **CVC**: Any 3 digits (4 for Amex)

---

## STC Pay

### Mobile Numbers

| Phone | Result |
|-------|--------|
| `0515555555` | Not registered |
| `0515555556` | Update required |
| `0515555557` | Invalid account |
| `0515555558` | OTP exhausted |
| `0515555559` | Wait 60 seconds |
| Any other | Proceeds to OTP |

### OTP Codes

| OTP | Result |
|-----|--------|
| `123456` | Success |
| `000000` | Success |
| `111111` | Insufficient balance |
| `222222` | Daily limit exceeded |
| `333333` | Max amount exceeded |
| `444444` | Timeout |
| Other | Invalid OTP |

---

## Apple Pay

Uses **amount ranges** (no test cards). Add real card to wallet, use test keys.

| Amount (SAR) | Result |
|--------------|--------|
| 200-300 | Success |
| 1000-1100 | Failure |
| 1101-1200 | Insufficient funds |
| 1201-1300 | Lost card |
| 1301-1400 | Declined |
| 1401-1500 | Expired card |
| 1501-1600 | Exceeds limit |
| 1601-1700 | Stolen card |

---

## Samsung Pay

Use `environment: 'STAGE'` and contact Samsung for test credentials.
