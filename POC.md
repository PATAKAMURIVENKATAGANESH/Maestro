# Maestro Accessibility Support - POC Status Report

## 📊 Executive Summary

| Platform | Status | Key Finding | Action Item |
|----------|--------|-------------|-------------|
| **Android** | ✅ **Complete** | `Configurator` pattern fixes TalkBack suppression. | PR Ready (`fix/accessibility-support`) |
| **iOS** | ✅ **Verified** | Maestro (standard) **does not suppress** VoiceOver. | No code changes needed. |

---

## 🤖 Android Investigation

### 🔴 Problem
Maestro's `UiAutomation` initialization was suppressing TalkBack audio feedback during test execution on Android 14 (API 34), even though accessibility services appeared "enabled" in system settings.

### 🟢 Solution Implemented
**The Configurator Pattern:**
We modified `MaestroDriverService.kt` to set the `uiAutomationFlags` **before** any `UiDevice` or `UiAutomation` instance is retrieved.

```kotlin
// MaestroDriverService.kt
Configurator.getInstance()
    .uiAutomationFlags = UiAutomation.FLAG_DONT_SUPPRESS_ACCESSIBILITY_SERVICES

val uiDevice = UiDevice.getInstance(instrumentation)
```

### ✅ Verification Results
- **TalkBack Audio:** Confirmed audible during test execution.
- **Service Coexistence:** TalkBack and BrowserStack Watcher remain active and bound.
- **Test Coverage:** Verified with:
    - Simple flows (taps)
    - Complex flows (15+ interactions, text input, scroll)
    - Demo App (`commands_tour.yaml`)

### 📝 Artifacts
- **Code:** `maestro-android/src/androidTest/java/dev/mobile/maestro/MaestroDriverService.kt`
- **Docs:** `accessibility_investigation/approach_1_flag_dont_suppress/README.md`

---

## 🍎 iOS Investigation

### ❓ Objective
Verify if Maestro on iOS suppresses VoiceOver or other accessibility services, similar to the Android issue.

### 🔍 Findings
1.  **Maestro Standard Driver:**
    - Used Maestro v2.0.10 CLI (standard distribution).
    - Driver: `dev.mobile.maestro-driver-iosUITests.xctrunner` (bundled).
    - **Result:** VoiceOver (`AccessibilityUIServer`) remains **active and visible** during test execution.
    - **No Suppression:** Logs show no evidence of accessibility services being disabled.

2.  **Test Execution:**
    - **Simulator:** iPhone 16 Plus (iOS 18.5).
    - **App:** Demo App (`com.example.example`).
    - **Flow:** Complex interactions (Text Input, Swipe, Repeat, Taps).
    - **Outcome:** All tests passed without timeouts. VoiceOver remained active.

3.  **Custom WDA (BrowserStack Utilities):**
    - **Status:** 🚧 In Progress / Blocked
    - **Goal:** Verify if custom WDA with BrowserStack utilities behaves differently.
    - **Current State:** Attempting to build WDA from `~/Documents/WebDriverAgent` with `~/wda-bstack-utilities`.
    - **Blocker:** Network timeouts installing Ruby dependencies (`xcodeproj`) needed for integration script.
    - **Observation:** Since standard Maestro WDA works, custom WDA is likely not strictly necessary for *fixing* accessibility, but might be needed for specific BrowserStack features.

### ✅ Conclusion
**Maestro on iOS supports accessibility out-of-the-box.** The suppression issue observed on Android does not exist on iOS with the standard Maestro driver.

### 📝 Artifacts
- **Test Flow:** `Maestro-iOS/e2e/ios_voiceover_test.yaml`
- **Logs:** `/tmp/ios_accessibility.log` (Analyzed)

---

## 🚀 Next Steps

1.  **Android:** Merge the PR `fix/accessibility-support`.
2.  **iOS:** Conclude verification. Standard Maestro is sufficient.
3.  **Custom WDA:** If strictly required, resolve network issues to build custom WDA (low priority for accessibility support itself).
