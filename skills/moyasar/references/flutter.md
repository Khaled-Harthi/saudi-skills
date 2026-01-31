# Flutter Integration

## Installation

```yaml
# pubspec.yaml
dependencies:
  moyasar: ^2.0.0
```

```bash
flutter pub get
```

**Android requirement:**
```gradle
// android/app/build.gradle
android {
    defaultConfig {
        minSdkVersion 21  // Required for 3DS WebView
    }
}
```

## Credit Card

```dart
import 'package:moyasar/moyasar.dart';
import 'package:uuid/uuid.dart';

class CheckoutPage extends StatelessWidget {
  final config = PaymentConfig(
    publishableApiKey: 'pk_test_...' // MOYASAR_PUBLISHABLE_KEY,
    amount: 10000,
    description: 'Order #123',
    metadata: {'order_id': '123'},
    givenId: Uuid().v4(),
  );

  void onPaymentResult(PaymentResult result) {
    if (result is PaymentResponse) {
      switch (result.status) {
        case PaymentStatus.paid:
          // Success - verify server-side before fulfilling
          break;
        case PaymentStatus.failed:
          // result.source.message - user-friendly error
          // result.source.responseCode - error code (51, 54, etc.)
          break;
        default:
          break;
      }
    } else if (result is TokenResponse) {
      // Handle token (if using saveCard flow)
    } else {
      // Handle errors
      if (result is NetworkEndpointError) {
        // API error
      } else if (result is NetworkError) {
        // Connection error
      } else if (result is GeneralError) {
        // General error
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return CreditCard(
      config: config,
      onPaymentResult: onPaymentResult,
    );
  }
}
```

## Apple Pay

Requires iOS setup: Enable Apple Pay capability in Xcode, configure merchant ID.

```dart
ApplePay(
  config: PaymentConfig(
    publishableApiKey: 'pk_test_...' // MOYASAR_PUBLISHABLE_KEY,
    amount: 10000,
    description: 'Order #123',
    applePay: ApplePayConfig(
      merchantId: 'merchant.com.yourapp',
      label: 'Your Store',
    ),
  ),
  onPaymentResult: handleResult,
)
```

## STC Pay

```dart
STCPay(
  config: config,
  onPaymentResult: handleResult,
)
```

## STC Pay

```dart
STCPay(
  config: PaymentConfig(
    publishableApiKey: 'pk_test_...' // MOYASAR_PUBLISHABLE_KEY
    amount: 10000,
    description: 'Order #123',
  ),
  onPaymentResult: (result) {
    if (result is PaymentResponse && result.status == PaymentStatus.paid) {
      // Success
    }
  },
)
```

**Test OTP:** `123456` for success, `000000` for failure.

## Custom UI

Build entirely custom forms using `Moyasar.pay()`:

```dart
// Collect card details in your custom form, then:
final result = await Moyasar.pay(
  apiKey: 'pk_test_...' // MOYASAR_PUBLISHABLE_KEY
  paymentRequest: PaymentRequest(
    amount: 10000,
    currency: 'SAR',
    description: 'Order #123',
    source: CreditCardSource(
      name: nameController.text,
      number: cardController.text,
      month: monthController.text,
      year: yearController.text,
      cvc: cvcController.text,
    ),
  ),
);

// Handle 3DS if result.status == PaymentStatus.initiated
// Redirect to result.source.transactionUrl
```

See [custom example](https://github.com/moyasar/moyasar-flutter/blob/main/example/lib/widgets/stc_ui_customization.dart).

## Key Points

- Check `result.status == PaymentStatus.paid` for success
- Use `givenId` for idempotency
- Still verify payment server-side after success
