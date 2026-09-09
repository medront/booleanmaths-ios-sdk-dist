# BooleanMaths iOS SDK

Analytics for iOS: event tracking, session handling, and per-install first-open
reporting, backed by a durable on-device queue that survives app restarts.

Distributed as a closed-source binary XCFramework. This repository hosts the
released builds — the SDK source is not public.

## Requirements

| | |
|---|---|
| Platform | iOS 17.0+ |
| Swift | 6.0 |
| Architectures | `arm64` (device), `arm64` + `x86_64` (simulator) |

## Installation

### CocoaPods

```ruby
platform :ios, '17.0'

target 'YourApp' do
  pod 'BooleanMathsSDK', '~> 1.0'
end
```

```sh
pod install
```

### Manual

Download `BooleanMathsSDK.xcframework.zip` from
[Releases](../../releases), unzip it, and drag `BooleanMathsSDK.xcframework`
into your target's **Frameworks, Libraries, and Embedded Content**, set to
*Embed & Sign*.

## Usage

Initialise once, as early as possible — typically in
`application(_:didFinishLaunchingWithOptions:)` or your `App` initialiser.

```swift
import BooleanMathsSDK

BooleanMaths.shared.initialize(
    apiKey: "your-api-key",
    pixelId: "your-pixel-id"
)
```

Track an event:

```swift
BooleanMaths.shared.track("product_viewed", properties: [
    "product_id": "SKU-1024",
    "price": 49.99
])
```

Flush the queue before a known interruption, such as a checkout hand-off:

```swift
BooleanMaths.shared.flush(timeout: 10) { success in
    // proceed regardless; queued events are retried on next launch
}
```

### API

| Member | Notes |
|---|---|
| `BooleanMaths.shared` | Singleton entry point. Main-actor isolated. |
| `initialize(apiKey:pixelId:)` | Call once. Subsequent calls are ignored. |
| `track(_:properties:)` | Queues an event. `properties` is optional. |
| `flush(timeout:completion:)` | Forces a dispatch attempt. Default timeout 30s. |
| `setWrapperConfig(type:version:)` | For cross-platform wrappers (React Native, Flutter). |
| `BooleanMaths.sdkVersion` | Reported with every event. |

Events are queued on disk and dispatched in batches. Nothing is lost if the app
is killed or the device is offline.

## Privacy

The SDK ships a `PrivacyInfo.xcprivacy` manifest declaring its data collection
and its use of `UserDefaults`. It collects a generated installation identifier,
device model and OS version, screen metrics, and the events you track. It does
**not** collect the advertising identifier (IDFA) or the vendor identifier
(IDFV).

You remain responsible for your app's own privacy disclosures, including its App
Store privacy labels, and for obtaining any consents required in your
jurisdiction. See §5 of the [LICENSE](LICENSE).

## License

Commercial. See [LICENSE](LICENSE).

Copyright © 2026 Medront Datalabs Private Limited.

## Support

saurav@booleanmaths.com
