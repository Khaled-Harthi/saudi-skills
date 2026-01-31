# iOS Integration

**Package:** `MoyasarSdk`

## Installation

**Swift Package Manager:**
```
File → Add Packages → https://github.com/moyasar/moyasar-ios-sdk
```

**CocoaPods:**
```ruby
pod 'MoyasarSdk', git: 'https://github.com/moyasar/moyasar-ios-pod.git'
```

## PaymentRequest

```swift
let request = PaymentRequest(
    apiKey: "pk_test_...", // MOYASAR_PUBLISHABLE_KEY
    amount: 10000,
    currency: "SAR",
    description: "Order #123",
    metadata: ["order_id": "123"],
    givenID: UUID().uuidString
)
```

## Components

| Component | Usage |
|-----------|-------|
| `CreditCardView` | Card payment form |
| `ApplePayView` | Apple Pay button |
| `STCPayView` | STC Pay flow |

```swift
CreditCardView(request: request) { result in
    handleResult(result)
}
```

## Result Handling

```swift
func handleResult(_ result: PaymentResult) {
    switch result {
    case .completed(let payment):
        if payment.status == "paid" {
            // Success - verify server-side
        } else if payment.status == "failed" {
            // payment.source.message - error message
            // payment.source.responseCode - error code
        }
    case .failed(let error):
        // SDK error
    case .canceled:
        // User canceled
    }
}
```

## Custom UI (Credit Card)

```swift
let service = PaymentService(apiKey: "pk_test_...")

let source = ApiCreditCardSource(
    name: "John Doe",
    number: "4111111111111111",
    month: "09",
    year: "25",
    cvc: "456",
    manual: "false",
    saveCard: "false"
)

let request = ApiPaymentRequest(
    amount: 1000,
    currency: "SAR",
    source: ApiPaymentSource.creditCard(source),
    metadata: ["sdk": "ios"]  // Required
)
```

## Custom UI (STC Pay)

```swift
let viewModel = STCPayViewModel(request: request) { result in
    handleResult(result)
}

viewModel.initiatePayment()  // Start payment
viewModel.submitOtp()        // Submit OTP
viewModel.screenStep         // Current screen
viewModel.isLoading          // Loading state
```

## Language

```swift
MoyasarLanguageManager.shared.language = .arabic  // .english, .system
```

## Key Points

- `PaymentResult.completed` ≠ paid — check `payment.status`
- Use `givenID` for idempotency
- Include `"sdk": "ios"` in metadata for custom UI
- Verify payment server-side
