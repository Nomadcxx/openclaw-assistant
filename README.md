# OpenClaw Assistant

![Build](https://github.com/Nomadcxx/openclaw-assistant/actions/workflows/ci.yml/badge.svg)

Security-hardened fork of [OpenClawAssistant](https://github.com/yuga-hashimoto/OpenClawAssistant). Android voice assistant that talks to a gateway server over TLS WebSocket. Provides node capabilities — SMS, camera, screen capture, location, canvas WebView — all remotely controllable.

This is a permanent fork. We maintain stricter security than upstream.

## Security

Compared to the original:

- TLS 1.2+ enforced on all connections (no fallback to older protocols)
- SMS rate limiting — 20/min global, 5/min per number
- WebView restricted to whitelisted URL schemes only
- Sensitive data stripped from production logs
- APK updates verified with SHA-256 hash + host validation
- Crypto keys and device identity excluded from Android cloud backup
- Reconnect retry capped to prevent infinite loops

Device identity uses Ed25519 via Tink + Android Keystore. All preferences encrypted with AES-256-GCM via `EncryptedSharedPreferences`. Gateway connections use TOFU TLS pinning with SHA-256 fingerprints.

Full audit: [docs/SECURITY-AUDIT.md](docs/SECURITY-AUDIT.md)

## Features

- Voice assistant via `VoiceInteractionService`
- Offline wake word + speech recognition (Vosk)
- Remote camera capture, screen recording, location tracking
- SMS send/receive
- Canvas WebView for interactive AI-driven UI
- mDNS/DNS-SD gateway discovery
- Cryptographic device pairing (Ed25519)

## Building

Requirements: JDK 17, Android SDK with API 35. Android Studio is optional but makes life easier.

```bash
git clone https://github.com/Nomadcxx/openclaw-assistant.git
cd openclaw-assistant
```

Firebase is used for Crashlytics but is not required. If you don't have a `google-services.json`, create a stub at `app/google-services.json`:

```json
{
  "project_info": { "project_number": "000000000000", "project_id": "stub", "storage_bucket": "stub.appspot.com" },
  "client": [{ "client_info": { "mobilesdk_app_id": "1:000000000000:android:0000000000000001", "android_client_info": { "package_name": "com.openclaw.assistant" } }, "api_key": [{ "current_key": "stub" }] }],
  "configuration_version": "1"
}
```

Build a debug APK:

```bash
./gradlew assembleDebug
```

The APK lands in `app/build/outputs/apk/debug/`. Install it with:

```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

For a signed release build, add your keystore details to `local.properties`:

```properties
storeFile=/path/to/your/release.keystore
storePassword=your_store_password
keyAlias=your_key_alias
keyPassword=your_key_password
```

Then:

```bash
./gradlew assembleRelease
```

Release APK goes to `app/build/outputs/apk/release/`.

## License

MIT
