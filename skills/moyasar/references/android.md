# Android Integration

**Package:** `com.github.Moyasar:moyasar-android-sdk:1.0.9`

## Installation

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        maven { url = uri("https://jitpack.io") }
    }
}

// app/build.gradle.kts
dependencies {
    implementation("com.github.Moyasar:moyasar-android-sdk:1.0.9")
}
```

**AndroidManifest.xml:**
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

## PaymentRequest

```kotlin
val request = PaymentRequest(
    apiKey = "pk_test_...", // MOYASAR_PUBLISHABLE_KEY
    amount = 10000,
    currency = "SAR",
    description = "Order #123",
    metadata = mapOf("order_id" to "123"),
    givenId = UUID.randomUUID().toString()
)
```

## Components

| Component | Usage |
|-----------|-------|
| `PaymentFragment` | Card payment form |
| `EnterMobileNumberFragment` | STC Pay flow |

```kotlin
val fragment = PaymentFragment.newInstance(application, request) { result ->
    handleResult(result)
}
```

## Result Handling

```kotlin
fun handleResult(result: PaymentResult) {
    when (result) {
        is PaymentResult.Completed -> {
            when (result.payment.status) {
                "paid" -> // Success - verify server-side
                "failed" -> {
                    // result.payment.source.message
                    // result.payment.source.responseCode
                }
            }
        }
        is PaymentResult.Failed -> // SDK error
        is PaymentResult.Canceled -> // User canceled
    }
}
```

## Restrict Networks

```kotlin
val request = PaymentRequest(
    // ...
    allowedNetworks = listOf(
        PaymentRequest.Network.VISA,
        PaymentRequest.Network.MASTERCARD,
        PaymentRequest.Network.MADA
    )
)
```

## Custom UI

Use `CreditCardViewModel` for custom card UI:

```kotlin
val viewModel = CreditCardViewModel(application, request)

viewModel.inputFieldsValidatorLiveData.observe(owner) { /* validation */ }
viewModel.creditCardStatus.observe(owner) { status ->
    when (status) {
        is CreditCardStatus.Success -> handleSuccess(status.payment)
        is CreditCardStatus.Error -> handleError(status.error)
        is CreditCardStatus.Reset3ds -> handle3ds(status.url)
    }
}

viewModel.submit(name, number, month, year, cvc)
```

For STC Pay: `EnterMobileNumberCustomUIFragment` with `stcPayStatus` observer.

## Key Points

- `PaymentResult.Completed` ≠ paid — check `payment.status`
- Use `givenId` for idempotency
- Verify payment server-side
