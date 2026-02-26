# OpenClaw Assistant — Security Audit Report

**Date**: 2026-02-26
**Auditor**: Automated Security Analysis
**Scope**: Full codebase review of com.openclaw.assistant Android application
**Commit**: 01678d9 (main branch)

---

## Executive Summary

The OpenClaw Assistant Android application implements a **strong security foundation** with Ed25519 device identity via Tink/Android Keystore, AES-256 encrypted preferences, TLS-only transport, and SHA-256 APK update verification. However, several **critical and high-severity vulnerabilities** were identified during this audit that require immediate remediation.

**Risk Profile**: 3 Critical, 4 High, 4 Medium, 2 Low findings.

---

## Findings

### 🔴 CRITICAL

#### C-1: Committed Firebase Debug Log (Data Exposure)
- **File**: `firebase-debug.log` (repository root)
- **Impact**: Exposes user email (`soccer.hy620@gmail.com`), Firebase project ID, OAuth2 token flows, GCP API patterns, and billing configuration in public repository history.
- **Fix**: Delete file, add to `.gitignore`, consider BFG Repo-Cleaner for history.
- **Status**: ✅ PATCHED

#### C-2: WebView Accepts Arbitrary URLs and JavaScript (RCE)
- **File**: `app/src/.../node/CanvasController.kt`
- **Lines**: `reload()` → `wv.loadUrl(currentUrl)`, `eval()` → `wv.evaluateJavascript(javaScript)`
- **Impact**: Gateway can inject `javascript:`, `file://`, `data:` scheme URLs or execute arbitrary JavaScript. If gateway is compromised, full device compromise is possible.
- **Fix**: URL scheme whitelist (`https://`, `file:///android_asset/` only), JavaScript source validation.
- **Status**: ✅ PATCHED

#### C-3: Cloud Backup Includes Sensitive Data
- **File**: `app/src/main/res/xml/backup_rules.xml`
- **Impact**: `<include domain="file" path="." />` backs up entire internal storage to Google cloud including Tink keystore files, device identity keys, and any cached auth data.
- **Fix**: Exclude sensitive directories and files from backup scope.
- **Status**: ✅ PATCHED

---

### 🟠 HIGH

#### H-1: No TLS Version Enforcement
- **File**: `app/src/.../gateway/GatewayTls.kt`
- **Lines**: Both `SSLContext.getInstance("TLS")` calls
- **Impact**: May negotiate TLSv1.0 or TLSv1.1 on older Android systems, enabling protocol downgrade attacks.
- **Fix**: Use `SSLContext.getInstance("TLSv1.2")` and configure enabled protocols.
- **Status**: ✅ PATCHED

#### H-2: Auth Flow Details Logged in Production
- **Files**: `GatewaySession.kt`, `DeviceIdentity.kt`, `ChatController.kt`
- **Impact**: `Log.d` calls without `BuildConfig.DEBUG` guards expose token/password auth attempts, device IDs, session keys, and idempotency keys in logcat. Any app on the device can read logcat on older Android versions.
- **Fix**: Wrap all sensitive log statements with `BuildConfig.DEBUG` checks.
- **Status**: ✅ PATCHED

#### H-3: No SMS Rate Limiting
- **File**: `app/src/.../node/SmsHandler.kt`, `SmsManager.kt`
- **Impact**: No per-number or per-time-window limits. A compromised gateway could use the device for SMS bombing or premium number fraud.
- **Fix**: Implement rate limiting (e.g., 5 SMS per number per minute, 20 total per minute).
- **Status**: ✅ PATCHED

#### H-4: Hardcoded GitHub URLs Point to Wrong Repository
- **Files**: `app/src/.../utils/UpdateChecker.kt`, `SettingsActivity.kt`
- **Impact**: Update checks and issue reporting point to `Nomadcxx/openclaw-assistant` instead of `Nomadcxx/openclaw-assistant`. Users could receive updates from untrusted source.
- **Fix**: Update all hardcoded GitHub URLs.
- **Status**: ✅ PATCHED

---

### 🟡 MEDIUM

#### M-1: APK Update Downloader Ignores Gateway TLS Config
- **File**: `app/src/.../node/AppUpdateHandler.kt`
- **Impact**: Creates separate OkHttpClient without certificate pinning for APK downloads. A MITM could serve malicious APK (though SHA-256 hash check mitigates).
- **Fix**: Pass gateway TLS config to download client.
- **Status**: ✅ PATCHED

#### M-2: Infinite Reconnect Loop
- **File**: `app/src/.../gateway/GatewaySession.kt`
- **Impact**: No max retry limit on WebSocket reconnection. On persistent failures, device battery drains from repeated connection attempts.
- **Fix**: Add configurable max retry limit with user notification.
- **Status**: ✅ PATCHED

#### M-3: Deprecated getParcelableExtra API
- **File**: `app/src/.../receiver/InstallResultReceiver.kt`
- **Impact**: Uses deprecated `getParcelableExtra(String)` instead of typed version for API 33+. Will cause warnings and may break in future Android versions.
- **Fix**: Use `getParcelableExtra(String, Class)` with SDK version check.
- **Status**: ✅ PATCHED

#### M-4: Predictable Temp File Names
- **File**: `app/src/.../node/ScreenRecordManager.kt`
- **Impact**: Uses predictable `openclaw-screen-` prefix for temp files. Theoretically enables local temp file prediction attacks (low practical risk on internal storage).
- **Fix**: Use `UUID.randomUUID()` in temp file names.
- **Status**: ✅ PATCHED

---

### 🟢 LOW / INFORMATIONAL

#### L-1: `.gitignore` Missing Log File Patterns
- **File**: `.gitignore`
- **Fix**: Add `*.log`, `firebase-debug.log` patterns.
- **Status**: ✅ PATCHED

#### L-2: PendingIntent FLAG_MUTABLE in AppUpdateHandler
- **File**: `app/src/.../node/AppUpdateHandler.kt`
- **Impact**: Uses `FLAG_MUTABLE` which is required by PackageInstaller API. Acceptable trade-off.
- **Status**: Accepted (by design)

---

## Positive Security Findings

| Feature | Assessment |
|---------|-----------|
| EncryptedSharedPreferences (AES256_SIV + AES256_GCM) | ✅ Properly implemented |
| Ed25519 device identity via Tink + Android Keystore | ✅ Strong key management |
| TLS fingerprint pinning (TOFU model) | ✅ Appropriate for IoT use case |
| SHA-256 hash verification for APK updates | ✅ Prevents tampering |
| HTTPS-only URL validation for updates | ✅ Scheme + host + credential checks |
| PendingIntent FLAG_IMMUTABLE usage | ✅ Used everywhere except PackageInstaller |
| Internal storage only for sensitive files | ✅ No world-readable files |
| `allowBackup=false` in AndroidManifest | ✅ Prevents legacy backup |
| `data_extraction_rules.xml` excludes prefs/db | ✅ Proper cloud exclusion |
| No exported components (except necessary BootReceiver) | ✅ Minimal attack surface |
| Network security config disables cleartext | ✅ Already fixed |

---

## Recommendations (Future Work)

1. **Security Test Suite**: Add unit tests for URL validation, rate limiting, TLS configuration
2. **Security Warning Dialog**: Show users what permissions the app requests and why
3. **StrictMode**: Enable in debug builds for detecting accidental disk/network on main thread
4. **Certificate Transparency**: Consider CT enforcement for gateway connections
5. **ProGuard Optimization**: Review keep rules for potential over-exposure
6. **Firebase Crashlytics Data**: Audit what exception context is sent to Firebase
