# Account Setup

## 1. Create an Account

Register at [authentica.sa](https://authentica.sa/). Verify your email and phone number.

## 2. Create an Application & Get API Key

1. Log in to the [Authentica portal](https://portal.authentica.sa/)
2. Go to **My Applications**
3. Click **Create New Application**
4. Enter the application name and check **SMS**
5. Click **Create Application**
6. Copy the API key
7. Store it as `AUTHENTICA_API_KEY` in your environment

## 3. Charge Credits

Each OTP costs credits. Top up via the billing section in the portal. Monitor your balance to avoid service interruption.

## 4. Register a Custom Sender ID

By default, OTPs use Authentica's default sender name. To use your own brand name:

1. Raise a sender ID request through the [Authentica portal](https://portal.authentica.sa/)
2. Provide **one** of the following legal documents:

| Document | Description |
|----------|-------------|
| **Commercial Registration** (سجل تجاري) | Proves legal ownership of the trade name (اسم تجاري). Sender name must match the registered name. |
| **Intellectual Property Certificate** (شهادة ملكية فكرية) | Issued by SAIP — Saudi Authority for Intellectual Property (الهيئة السعودية للملكية الفكرية). Proves trademark ownership for the sender name. |

- The sender name must exactly match the name in your legal documentation.
- Review takes time — use the default sender while waiting.
- Once approved, configure it as default in **Application Settings** on the dashboard.
