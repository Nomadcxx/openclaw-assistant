# AGENTS.md — OpenClaw Assistant

## Project Overview

**OpenClaw Assistant** is an Android Kotlin application that acts as a remote-controllable voice assistant with gateway communication over TLS WebSocket. It provides node capabilities including SMS, camera, screen capture, location tracking, and a canvas WebView.

- **Language**: Kotlin (100%)
- **Platform**: Android (minSdk 31, targetSdk 34, compileSdk 35)
- **Build System**: Gradle (Kotlin DSL) with AGP 8.6.1
- **UI Framework**: Jetpack Compose (BOM 2024.09.03) + Material 3
- **Architecture**: Single-module (`app/`) with package-based organization

## Architecture

```
com.openclaw.assistant/
├── api/              # HTTP API client (OpenClawClient, OkHttp)
├── camera/           # Camera management (CameraX 1.5.2)
├── chat/             # Chat models, controller, markdown
├── data/             # Room DB, DAOs, repositories, entities
├── gateway/          # TLS, auth, device identity (Tink + BouncyCastle Ed25519),
│                     #   service discovery (mDNS/DNS-SD), WebSocket protocol
├── node/             # Node handlers: SMS, Camera, Screen, Location, Canvas,
│                     #   App Update, Debug, Connection management
├── protocol/         # Canvas A2UI actions, protocol constants
├── receiver/         # Boot receiver, install result receiver
├── service/          # Foreground services, hotword, voice session
├── speech/           # TTS, speech recognition (Vosk 0.3.75), diagnostics
├── tools/            # Tool display utilities
├── ui/               # Compose UI: theme, screens, components, camera HUD
└── utils/            # Device names, gateway config, system info, update checker
```

## Key Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| OkHttp | 4.12.0 | HTTP/WebSocket client |
| Tink | 1.10.0 | Cryptographic operations, key management |
| BouncyCastle | 1.83 | Ed25519 device identity signatures |
| Room | 2.6.1 | Local SQLite database |
| CameraX | 1.5.2 | Camera capture |
| Vosk | 0.3.75 | Offline speech recognition |
| Firebase | BOM 32.7.4 | Crashlytics + Analytics (configurable) |
| dnsjava | 3.6.4 | DNS-SD service discovery |
| Gson | — | JSON serialization |
| kotlinx-serialization | — | JSON parsing |

## Security Architecture

### Cryptography
- **Device Identity**: Ed25519 key pair via Tink, stored in Android Keystore
- **Preferences**: EncryptedSharedPreferences (AES256_SIV keys + AES256_GCM values)
- **Auth Protocol**: Signed nonce-challenge with deviceId, clientId, role, scopes, timestamp

### Network
- **Transport**: TLS WebSocket to gateway
- **Trust Model**: TOFU (Trust On First Use) with SHA-256 fingerprint pinning
- **Cleartext**: Disabled via network_security_config.xml
- **APK Updates**: HTTPS-only with SHA-256 hash verification, host validation

### Sensitive Permissions
`RECORD_AUDIO`, `CAMERA`, `FINE_LOCATION`, `BACKGROUND_LOCATION`, `SEND_SMS`, `REQUEST_INSTALL_PACKAGES`, `MEDIA_PROJECTION`

## Build & Run

```bash
# Debug build
./gradlew assembleDebug

# Release build (requires signing config in local.properties)
./gradlew assembleRelease

# Run tests
./gradlew test

# Lint
./gradlew lint
```

## Testing

Minimal test coverage — only 2 test files:
- `HotwordServiceTest.kt`
- `SessionFiltersTest.kt`

## CI/CD

GitHub Actions workflows:
- `ci.yml` — Build validation
- `build-release.yml` — Release APK build
- `jules-from-issue-label.yml` — Automated issue handling

## Repository

- **Origin**: https://github.com/Nomadcxx/openclaw-assistant
- **License**: MIT
- **Upstream Fork**: yuga-hashimoto/OpenClawAssistant

## Development Guidelines

1. **Security First**: All network communication must use TLS. No cleartext traffic.
2. **Encrypted Storage**: Sensitive data must use EncryptedSharedPreferences or Android Keystore.
3. **Production Logging**: Never log tokens, passwords, keys, or auth flow details. Use `BuildConfig.DEBUG` guards.
4. **Input Validation**: Validate all gateway-provided URLs, parameters, and commands.
5. **Minimal Permissions**: Request only required permissions, validate at runtime.
6. **Backup Exclusion**: Sensitive files must be excluded from cloud backup via data_extraction_rules.xml.
