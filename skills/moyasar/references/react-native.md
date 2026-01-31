# React Native Integration

## Installation

```bash
npm install react-native-moyasar-sdk react-native-webview react-native-svg
cd ios && pod install
```

**Android (for Samsung Pay):**
```gradle
// android/app/build.gradle
android {
    buildFeatures {
        dataBinding true
    }
}
```

## Imports

```tsx
import {
  ApplePay,
  ApplePayConfig,
  CreditCard,
  CreditCardConfig,
  PaymentConfig,
  PaymentResult,
  PaymentResponse,
  PaymentStatus,
  createPayment,
  CreditCardRequestSource,
  CreditCardResponseSource,
  PaymentRequest,
  NetworkError,
  NetworkEndpointError,
  GeneralError,
  WebviewPaymentAuth,  // For 3DS handling
} from 'react-native-moyasar-sdk';

// Card brand logos (for custom UI)
import { Mada } from 'react-native-moyasar-sdk/src/assets/mada';
import { Visa } from 'react-native-moyasar-sdk/src/assets/visa';
import { Mastercard } from 'react-native-moyasar-sdk/src/assets/mastercard';
```

## Credit Card

```tsx
import uuid from 'react-native-uuid';

const paymentConfig = new PaymentConfig({
  givenId: uuid.v4() as string,  // Idempotency
  publishableApiKey: 'pk_test_...', // MOYASAR_PUBLISHABLE_KEY
  amount: 10000,  // halalas (100.00 SAR)
  currency: 'SAR',
  description: 'Order #123',
  metadata: { order_id: '123', user_id: '456' },
  supportedNetworks: ['mada', 'visa', 'mastercard', 'amex'],
  creditCard: new CreditCardConfig({ saveCard: false, manual: false }),
});

function CheckoutScreen() {
  const onPaymentResult = (result: PaymentResult) => {
    if (result instanceof PaymentResponse) {
      if (result.status === PaymentStatus.paid) {
        // Success - verify server-side before fulfilling
      } else if (result.status === PaymentStatus.failed) {
        // result.source.message - user-friendly error
        // result.source.responseCode - error code (51, 54, etc.)
      } else if (result.status === PaymentStatus.initiated) {
        // 3DS required - SDK handles automatically for built-in components
      }
    } else if (result instanceof NetworkError || result instanceof NetworkEndpointError) {
      // Network error - ask user to check connection
    } else if (result instanceof GeneralError) {
      // General error
    }
  };

  return (
    <CreditCard
      paymentConfig={paymentConfig}
      onPaymentResult={onPaymentResult}
    />
  );
}
```

## Apple Pay

```tsx
const paymentConfig = new PaymentConfig({
  givenId: uuid.v4() as string,
  publishableApiKey: 'pk_test_...', // MOYASAR_PUBLISHABLE_KEY
  amount: 10000,
  currency: 'SAR',
  description: 'Order #123',
  applePay: new ApplePayConfig({
    merchantId: 'merchant.com.yourapp',
    label: 'Your Store',
    manual: false,
    saveCard: false,
  }),
});

<ApplePay
  paymentConfig={paymentConfig}
  onPaymentResult={onPaymentResult}
  style={{ buttonType: 'pay', buttonStyle: 'black' }}
/>
```

## STC Pay

```tsx
import { StcPay } from 'react-native-moyasar-sdk';

<StcPay paymentConfig={paymentConfig} onPaymentResult={onPaymentResult} />
```

## Samsung Pay

```tsx
import { SamsungPay } from 'react-native-moyasar-sdk';

const config = new PaymentConfig({
  // ... base config
  samsungPay: {
    serviceId: 'your_service_id',
    merchantName: 'Your Store',
  },
});

<SamsungPay paymentConfig={config} onPaymentResult={onPaymentResult} />
```

## UI Customization

Style the built-in components:

```tsx
<CreditCard
  paymentConfig={paymentConfig}
  onPaymentResult={onPaymentResult}
  style={{
    container: { backgroundColor: '#f5f5f5' },
    textInputs: { borderWidth: 1.25, color: 'black' },
    textInputsPlaceholderColor: 'gray',
    paymentButton: { borderRadius: 8 },
    paymentButtonText: { fontSize: 16 },
    errorText: { color: 'red' },
    activityIndicatorColor: '#007AFF',
  }}
/>

<ApplePay
  paymentConfig={paymentConfig}
  buttonType="buy"  // 'buy' | 'plain' | 'pay' | 'inStore' | 'donate' | 'setUp'
  buttonStyle="black"  // 'black' | 'white' | 'whiteOutline'
  height={50}
  cornerRadius={8}
/>
```

## Custom UI (Build Your Own)

Full control with `createPayment()`:

```tsx
// 1. Create source with card details from your custom form
const source = new CreditCardRequestSource({
  name: cardDetails.name,
  number: cardDetails.number,
  cvc: cardDetails.cvc,
  month: cardDetails.expiryMonth,
  year: cardDetails.expiryYear,
  tokenizeCard: false,
  manualPayment: false,
});

// 2. Create payment request
const request = new PaymentRequest({
  givenId: paymentId,
  amount: 10000,  // halalas
  currency: 'SAR',
  description: 'Order #123',
  metadata: { order_id: '123' },
  source,
  callbackUrl: 'https://sdk.moyasar.com/return',  // Default SDK callback
});

// 3. Process payment
const payment = await createPayment(request, 'pk_test_...'); // MOYASAR_PUBLISHABLE_KEY

// 4. Handle 3DS if needed
if (payment instanceof PaymentResponse) {
  if (payment.status === PaymentStatus.initiated) {
    // 3DS required - get transaction URL
    if (payment.source instanceof CreditCardResponseSource) {
      const transactionUrl = payment.source.transactionUrl;
      // Navigate to WebviewPaymentAuth
    }
  } else if (payment.status === PaymentStatus.paid) {
    // Success without 3DS
  }
}
```

## 3DS WebView Handling

For custom UI, handle 3DS authentication with `WebviewPaymentAuth`:

```tsx
import { WebviewPaymentAuth } from 'react-native-moyasar-sdk';

function PaymentWebView({ transactionUrl }) {
  const handleResult = (response) => {
    const { status, message } = response;

    if (status === 'paid' || status === 'authorized') {
      // Success - navigate to success screen
    } else if (status === 'failed') {
      // Failed - show error message
    }
  };

  return (
    <WebviewPaymentAuth
      transactionUrl={transactionUrl}
      onWebviewPaymentAuthResult={handleResult}
      callbackUrl="https://sdk.moyasar.com/return"
      style={{ webviewActivityIndicatorColor: '#007AFF' }}
    />
  );
}
```

## Key Points

- Use `PaymentStatus.paid` enum instead of string comparison
- Use `instanceof` checks for proper type narrowing
- `WebviewPaymentAuth` handles 3DS for custom UI flows
- Default callback URL: `https://sdk.moyasar.com/return`
- Card brand assets available from SDK for custom UI
- Always verify payment server-side after success
