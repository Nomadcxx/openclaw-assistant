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

```bash
./gradlew assembleDebug
./gradlew assembleRelease   # requires signing config in local.properties
```

## License

MIT
