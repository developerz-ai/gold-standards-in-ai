# 📱 Mobile — Native Swift + Kotlin

For mobile, go **native**: Swift (iOS) and Kotlin (Android). Best platform UX, full access to platform APIs, no cross-platform abstraction tax. Both share **one backend** ([Bun/Hono](backend-bun-hono.md) or [Rust](rust-apis.md)) — the API is the contract.

## Why native over cross-platform
- Platform-native look, feel, performance, and APIs.
- No bridge layer to debug; crashes map to real stack traces.
- The shared backend already enforces the domain — the apps stay thin clients.

## Repo shape (in the monorepo)
```
mobile/
├── <app>/
│   ├── ios/        # SwiftUI app
│   └── android/    # Jetpack Compose app
└── shared/
    ├── ios/        # Swift Package (SPM) — shared client, models, i18n
    └── android/    # Kotlin module — shared client, models, i18n
```
Per-platform "shared" packages hold the API client, DTOs, and locale handling so screens stay thin.

## iOS — SwiftUI
```swift
import SwiftUI
import AppShared

@main
struct MyApp: App {
    init() {
        Telemetry.start(appType: "myapp")   // crash reporting before any view
    }
    var body: some Scene { WindowGroup { ContentView() } }
}
```
Shared code as a Swift Package:
```swift
let package = Package(
  name: "AppShared",
  platforms: [.iOS(.v15)],
  products: [.library(name: "AppShared", targets: ["AppShared"])],
  targets: [
    .target(name: "AppShared"),
    .testTarget(name: "AppSharedTests", dependencies: ["AppShared"]),
  ]
)
```

## Android — Jetpack Compose + Ktor
```kotlin
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent { AppTheme { Surface(Modifier.fillMaxSize()) { AppNavHost() } } }
  }
}
```
Shared API client with Ktor:
```kotlin
class Api(private val baseUrl: String, http: HttpClient? = null) {
  private val client = http ?: HttpClient(OkHttp) {
    install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
  }
  suspend fun day(date: String, locale: String): DailyReading =
    client.get("$baseUrl/day/$date") { parameter("locale", locale) }.body()
}
```

## Shared conventions across both
- **Locale passed per request** (`?locale=es`); normalize tags client-side (`pt-BR` → `pt`, fallback `en`). Same locale set the web uses → [../frontend-craft/i18n.md](../frontend-craft/i18n.md).
- **One backend contract.** Version the API; don't fork logic into the apps.
- **Crash/error reporting** wired before the first view mounts, per-app key.
- **Tests:** `XCTest` (iOS), JUnit (Android) on the shared modules at minimum.
- An umbrella web site can tie the apps together for marketing/onboarding ([SSR Solid](frontend-solidjs.md)).
