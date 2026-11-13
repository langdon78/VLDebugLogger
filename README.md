# VL (Very Light 🪶) Debug Logger

A small structured logger for Swift, used by the other VL packages ([VLNetworkingClient](https://github.com/langdon78/VLNetworkingClient), [VLOAuthFlowCoordinator](https://github.com/langdon78/VLOAuthFlowCoordinator) and [VLDiscogsClient](https://github.com/langdon78/VLDiscogsClient)). It writes to the console, to `os.Logger`, or both.

## Requirements

- iOS 16+ / macOS 13+
- Swift 6.2+

## Installation

```swift
dependencies: [
    .package(url: "https://github.com/langdon78/VLDebugLogger", .upToNextMinor(from: "0.2.1"))
]
```

## Usage

`VLDebugLogger.shared` is on by default. Any instance you create yourself starts off and logs nothing until you turn it on.

```swift
import VLDebugLogger

let logger = VLDebugLogger(subsystem: "com.example.myapp", enabled: true)

logger.log("Starting sync", category: .network, level: .info)
logger.log(error, message: "Sync failed", category: .network)

logger.toggleEnabled(to: false)
```

Messages are filtered by level (`logLevels`) and sent to `destination` (`.console`, `.osLog`). Categories are `.oauth`, `.keychain`, `.network`, `.general`, `.error`, or `.custom(customCategory:)`. The order of the `OrderedSet<MessageFormatOption>` you pass as `format` (or `defaultFormat`) sets the order of the parts in each line.

### Requests and responses

```swift
logger.log(request)                          // method, URL, headers and body
logger.log(response, data: data)             // status code, URL and body size
logger.log(response, data: data, showData: true)
```

`log(_ request:)` leaves out the `Authorization` and `X-API-Key` headers. Any other credential header is logged as is. `showData: true` prints the whole response body, so don't use it for responses that can carry tokens or personal data.

## License

MIT. See [LICENSE](LICENSE).
