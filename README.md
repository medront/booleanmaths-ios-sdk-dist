# BooleanMaths iOS SDK

Analytics for iOS: event tracking, session handling, and per-install first-open
reporting, backed by a durable on-device queue that survives app restarts.

Distributed as a closed-source binary XCFramework. This repository hosts the
released builds — the SDK source is not public.

## Requirements

| | |
|---|---|
| Platform | iOS 15.0+ |
| Swift | 6.0 |
| Architectures | `arm64` (device), `arm64` + `x86_64` (simulator) |

## Installation

### CocoaPods

```ruby
platform :ios, '15.0'

target 'YourApp' do
  pod 'BooleanMathsSDK', '~> 1.2'
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

### Development vs production

`initialize` takes an optional `isDebug` flag (1.1.0+). Every event the SDK
sends reports an `environment` of `"development"` when it is true and
`"production"` when it is false, so test traffic can be separated from real
traffic in reporting.

```swift
#if DEBUG
let isDebug = true
#else
let isDebug = false
#endif

BooleanMaths.shared.initialize(
    apiKey: "your-api-key",
    pixelId: "your-pixel-id",
    isDebug: isDebug
)
```

The flag defaults to `false`, so omitting it reports production. It is recorded
on each event as it is tracked, not when it is sent — an event queued by a debug
build still reports `"development"` even if it is delivered after the user
updates to a release build.

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
| `initialize(apiKey:pixelId:isDebug:)` | Call once. Subsequent calls are ignored. `isDebug` defaults to `false`. |
| `track(_:properties:)` | Queues an event. `properties` is optional. |
| `flush(timeout:completion:)` | Forces a dispatch attempt. Default timeout 30s. |
| `setWrapperConfig(type:version:)` | For cross-platform wrappers (React Native, Flutter). |
| `BooleanMaths.sdkVersion` | Reported with every event. |

Events are queued on disk and dispatched in batches. Nothing is lost if the app
is killed or the device is offline.

## Privacy

The SDK ships a `PrivacyInfo.xcprivacy` manifest declaring its data collection
and its use of `UserDefaults`. It collects a generated installation identifier,
device model and OS version, screen metrics, your app's bundle identifier,
version and install/update dates, and the events you track. It does **not**
collect the advertising identifier (IDFA) or the vendor identifier (IDFV), and
it does not read file timestamps or any other required-reason API beyond
`UserDefaults`.

You remain responsible for your app's own privacy disclosures, including its App
Store privacy labels, and for obtaining any consents required in your
jurisdiction. See §5 of the [LICENSE](LICENSE).

## License

Commercial. See [LICENSE](LICENSE).

Copyright © 2026 Medront Datalabs Private Limited.

## Support

saurav@booleanmaths.com
