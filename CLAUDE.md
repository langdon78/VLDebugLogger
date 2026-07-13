# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A small, single-file Swift Package providing structured debug logging (`VLDebugLogger`), used across the `VL*` family of libraries (`VLNetworkingClient`, `VLOAuthFlowCoordinator`, and transitively `VLDiscogsClient`). Depends only on `swift-collections` (for `OrderedSet` in format configuration).

## Commands

```sh
swift build
swift test
swift test --filter VLDebugLoggerTests
```

## Architecture

Everything lives in `Sources/VLDebugLogger/VLDebugLogger.swift` — a single `final class VLDebugLogger: Sendable`.

- **`VLDebugLogger.shared`** — a pre-configured static instance with `enabled: true`. Any other instance created via `init(...)` defaults to `enabled: false` — logging is opt-in per instance, not globally on.
- Logging is gated by both `enabled` and whether the message's `LogLevel` is in the instance's `logLevels` set — both must pass or the call is a no-op.
- **Destinations**: `.console` (via `print`) and `.osLog` (via `os.log`'s `Logger`, only on platforms where `os` is importable — the type falls back gracefully via `#if canImport(os)` elsewhere, e.g. Linux).
- **Categories**: `.oauth`, `.keychain`, `.network`, `.general`, `.error`, or `.custom(customCategory:)`.
- **Format is configurable per-call or per-instance** via `OrderedSet<MessageFormatOption>` (`.prefix`, `.message`, `.subsystem`, `.category`, `.emoji`, `.error`) — order in the set determines the order components are joined in the output line.

### Request/response logging redacts sensitive headers automatically

`log(_ request: URLRequest)` calls `logRequestHeaders`, which explicitly filters out `Authorization` and `X-API-Key` headers before logging (`Sources/VLDebugLogger/VLDebugLogger.swift`, `logRequestHeaders`). This is a real security property consumers rely on implicitly — if you add a new kind of credential header elsewhere in the `VL*` stack (a different header name for a different service), it will **not** be redacted by this filter unless it's added to that explicit exclusion list. Don't assume all sensitive headers are automatically safe to log through this path — verify the header name is actually in the filter.

Response body logging (`log(_ response:data:showData:)`) does **not** redact anything in the body itself — `showData: true` will print the full raw payload. Treat `showData: true` as unsafe for anything that might contain tokens or PII in the response body, not just headers.
