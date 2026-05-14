# 🔐 Google Maps API Key — Secure Setup Guide

> Securely load your Google Maps API key in a Flutter project  
> using `.env` (Android) and Xcode Build Settings (iOS)  
> without hardcoding secrets anywhere in source code.

---

# ✨ Why This Setup?

Hardcoding API keys is unsafe.

| File | Risk |
|------|------|
| `AndroidManifest.xml` | Exposed after APK decompile |
| `build.gradle.kts` | Exposed in source |
| `Info.plist` | Exposed in IPA inspection |
| `AppDelegate.swift` | Exposed in source |

This setup keeps your key out of GitHub and source code.

---

# 🔄 Secure Key Flow

## 🤖 Android

```text
.env (gitignored)
        ↓
build.gradle.kts
        ↓
AndroidManifest.xml
        ↓
Google Maps SDK Runtime
```

---

## 🍎 iOS

```text
Xcode Build Settings
        ↓
Info.plist
        ↓
AppDelegate.swift
        ↓
Google Maps SDK Runtime
```

---

# 📦 Step 1 — Create `.env`

Create a `.env` file in the project root:

```env
GOOGLE_MAPS_SDK_KEY=AIzaSy..........................
```

Add this to `.gitignore`:

```gitignore
.env
android/local.properties
android/key.properties
```

---

# 🤖 Android Setup

## 📁 `android/app/build.gradle.kts`

```kotlin
val envProperties = Properties()
val envFile = rootProject.file("../.env")

if (envFile.exists()) {
    envProperties.load(FileInputStream(envFile))
}

defaultConfig {
    manifestPlaceholders["GOOGLE_MAPS_SDK_KEY"] =
        envProperties.getProperty("GOOGLE_MAPS_SDK_KEY")
            ?: System.getenv("GOOGLE_MAPS_SDK_KEY")
            ?: ""
}
```

---

## 📄 `AndroidManifest.xml`

❌ Remove:

```xml
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="AIzaSy.........................." />
```

✅ Add:

```xml
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="${GOOGLE_MAPS_SDK_KEY}" />
```

---

# 🍎 iOS Setup

## 🛠 Add User-Defined Setting in Xcode

Open:

```text
ios/Runner.xcworkspace
```

Go to:

```text
Runner
 └── TARGETS
      └── Runner
           └── Build Settings
```

Then:

- Select **All**
- Select **Combined**
- Click **+**
- Choose **Add User-Defined Setting**

Add:

```text
GOOGLE_MAPS_SDK_KEY
```

Set values for:

```text
Debug
Profile
Release
```

---

## 📄 `Info.plist`

❌ Remove:

```xml
<key>GMSApiKey</key>
<string>AIzaSy..........................</string>
```

✅ Add:

```xml
<key>GMSApiKey</key>
<string>$(GOOGLE_MAPS_SDK_KEY)</string>
```

---

## 🚀 `AppDelegate.swift`

```swift
import Flutter
import UIKit
import GoogleMaps

@main
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {

  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {

    if let mapsApiKey = Bundle.main.object(
      forInfoDictionaryKey: "GMSApiKey"
    ) as? String {

      GMSServices.provideAPIKey(mapsApiKey)
    }

    return super.application(
      application,
      didFinishLaunchingWithOptions: launchOptions
    )
  }

  func didInitializeImplicitFlutterEngine(
    _ engineBridge: FlutterImplicitEngineBridge
  ) {
    GeneratedPluginRegistrant.register(
      with: engineBridge.pluginRegistry
    )
  }
}
```

---

# ⚙️ GitHub Actions CI/CD

## 🔑 Add GitHub Secret

Go to:

```text
GitHub Repo
 └── Settings
      └── Secrets and variables
           └── Actions
```

Create:

```text
GOOGLE_MAPS_SDK_KEY
```

---

## 🤖 Android Workflow

```yaml
- name: Inject Maps API Key into .env
  run: echo "GOOGLE_MAPS_SDK_KEY=${{ secrets.GOOGLE_MAPS_SDK_KEY }}" >> .env
```

---

## 🍎 iOS Workflow

```yaml
- name: Inject Maps API Key into Info.plist
  run: |
    /usr/libexec/PlistBuddy -c \
      "Set :GMSApiKey ${{ secrets.GOOGLE_MAPS_SDK_KEY }}" \
      ios/Runner/Info.plist
```

---

# ✅ Verify Setup

## 🤖 Android

```bash
flutter clean
flutter pub get
flutter run
```

If the map shows grey tiles:

- Verify `.env` exists
- Verify key name is exactly:

```text
GOOGLE_MAPS_SDK_KEY
```

---

## 🍎 iOS

```bash
flutter clean
flutter pub get
cd ios && pod install && cd ..
flutter run
```

If the app crashes:

- Verify `GOOGLE_MAPS_SDK_KEY`
  exists in:
  - Debug
  - Profile
  - Release

---

# 🔒 Best Practices

✅ Never hardcode keys  
✅ Never commit `.env`  
✅ Use GitHub Secrets in CI/CD  
✅ Restrict API key in Google Cloud Console  
✅ Enable only required APIs

---

# 🧠 Complete Architecture

## Android

```text
.env
   ↓
build.gradle.kts
   ↓
AndroidManifest.xml
   ↓
Google Maps SDK
```

---

## iOS

```text
Xcode Build Settings
   ↓
Info.plist
   ↓
AppDelegate.swift
   ↓
Google Maps SDK
```

---

# ❤️ Result

Your Flutter app now uses:

- Secure API key injection
- Git-safe secrets
- CI/CD compatible configuration
- Production-ready Google Maps setup

---
