Google Maps API Key — Secure Setup Guide
This guide explains how to securely load the Google Maps API key in your Flutter project
using a .env file (Android) and Xcode Build Settings (iOS) — no hardcoded keys anywhere.

The Problem
Your API key must never be hardcoded in any file committed to Git.
FileRiskandroid/app/src/main/AndroidManifest.xmlExposed after APK decompileandroid/app/build.gradle.ktsExposed in source codeios/Runner/Info.plistExposed in IPA inspectionios/Runner/AppDelegate.swiftExposed in source code

The Secure Chain
ANDROID
.env (git-ignored)  →  build.gradle.kts  →  AndroidManifest.xml  →  Runtime

iOS
Xcode Build Settings (never committed)  →  Info.plist  →  AppDelegate.swift  →  Runtime

Step 1 — .env File (Project Root)
Make sure your .env file has this exact key name:
envGOOGLE_MAPS_SDK_KEY=AIzaSy..........................
Make sure .env is in your .gitignore:
gitignore.env
android/local.properties
android/key.properties

Step 2 — Android: android/app/build.gradle.kts
Your build.gradle.kts already has the correct setup — no changes needed.
kotlin// Already in your build.gradle.kts — no changes needed
val envProperties = Properties()
val envFile = rootProject.file("../.env")
if (envFile.exists()) {
    envProperties.load(FileInputStream(envFile))
}

defaultConfig {
    manifestPlaceholders["GOOGLE_MAPS_SDK_KEY"] =
        envProperties.getProperty("GOOGLE_MAPS_SDK_KEY")   // 1st: from .env
            ?: System.getenv("GOOGLE_MAPS_SDK_KEY")         // 2nd: from CI/CD env
            ?: ""                                            // 3rd: empty fallback
}

Step 3 — Android: AndroidManifest.xml
Replace the hardcoded key with the placeholder variable:
xml<!-- REMOVE this -->
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="AIzaSy.........................."/>

<!-- ADD this -->
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="${GOOGLE_MAPS_SDK_KEY}"/>

Step 4 — iOS: Add the Key in Xcode

This is the iOS equivalent of your .env file.
The key lives only inside Xcode — never in any file pushed to Git.

Open your project
Open Xcode → Runner.xcworkspace (always open .xcworkspace, never .xcodeproj)
Add the key as a User-Defined Setting
1. Left sidebar
   └── Click the Runner project (top-level blue icon)

2. Under TARGETS
   └── Click Runner

3. Click the "Build Settings" tab

4. At the top of Build Settings
   └── Select "All" and "Combined"

5. Click the "+" button (top-left of the Build Settings list)
   └── Choose "Add User-Defined Setting"

6. A new row appears at the bottom under "User-Defined"
   └── Type the name exactly: GOOGLE_MAPS_SDK_KEY
   └── Press Enter

7. Click the arrow (▶) on the left of the row to expand it
   └── Set the value for each configuration:

       Debug   →  AIzaSy..........................
       Profile →  AIzaSy..........................
       Release →  AIzaSy..........................

8. Press Cmd+S to save
Now $(GOOGLE_MAPS_SDK_KEY) in Info.plist will automatically resolve to your
key value at build time for all configurations.

Step 5 — iOS: Info.plist
Replace the hardcoded key:
xml<!-- REMOVE this -->
<key>GMSApiKey</key>
<string>AIzaSy..........................</string>

<!-- ADD this — resolves from Xcode Build Settings at build time -->
<key>GMSApiKey</key>
<string>$(GOOGLE_MAPS_SDK_KEY)</string>

Step 6 — iOS: AppDelegate.swift
Replace the hardcoded key with a secure read from Info.plist:
swiftimport Flutter
import UIKit
import GoogleMaps

@main
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {

    // Key is read from Info.plist which resolves $(GOOGLE_MAPS_SDK_KEY)
    // from Xcode Build Settings — never hardcoded here
    if let mapsApiKey = Bundle.main.object(forInfoDictionaryKey: "GMSApiKey") as? String {
      GMSServices.provideAPIKey(mapsApiKey)
    }

    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  func didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge) {
    GeneratedPluginRegistrant.register(with: engineBridge.pluginRegistry)
  }
}

Step 7 — CI/CD: GitHub Actions
Add the secret to GitHub
1. Go to your GitHub repo
2. Click Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Name:  GOOGLE_MAPS_SDK_KEY
5. Value: AIzaSy..........................
6. Click Add secret
Android — inject into .env in your workflow .yml
yaml- name: Inject Maps API Key into .env
  run: echo "GOOGLE_MAPS_SDK_KEY=${{ secrets.GOOGLE_MAPS_SDK_KEY }}" >> .env
iOS — inject into Info.plist in your workflow .yml
yaml- name: Inject Maps API Key into Info.plist
  run: |
    /usr/libexec/PlistBuddy -c \
      "Set :GMSApiKey ${{ secrets.GOOGLE_MAPS_SDK_KEY }}" \
      ios/Runner/Info.plist

Step 8 — Verify
Android
bashflutter clean
flutter pub get
flutter run
If the map shows grey tiles → the key was not injected.
Check that the key name in .env is exactly GOOGLE_MAPS_SDK_KEY (case-sensitive).
iOS
bashflutter clean
flutter pub get
cd ios && pod install && cd ..
flutter run
If the app crashes on launch → verify GOOGLE_MAPS_SDK_KEY exists in Xcode Build
Settings under User-Defined for all three configurations (Debug / Profile / Release).

Complete Key Flow Reference
ANDROID
────────────────────────────────────────────────────────
.env
  GOOGLE_MAPS_SDK_KEY=AIzaSy..........................
        │
        ▼
build.gradle.kts
  envProperties.getProperty("GOOGLE_MAPS_SDK_KEY")
  manifestPlaceholders["GOOGLE_MAPS_SDK_KEY"] = ...
        │
        ▼
AndroidManifest.xml
  android:value="${GOOGLE_MAPS_SDK_KEY}"
        │
        ▼
Runtime — Google Maps SDK initializes with the key


iOS
────────────────────────────────────────────────────────
Xcode Build Settings → User-Defined
  GOOGLE_MAPS_SDK_KEY = AIzaSy..........................
        │
        ▼
Info.plist
  <string>$(GOOGLE_MAPS_SDK_KEY)</string>
        │
        ▼
AppDelegate.swift
  Bundle.main.object(forInfoDictionaryKey: "GMSApiKey")
        │
        ▼
GMSServices.provideAPIKey(mapsApiKey)
        │
        ▼
Runtime — Google Maps SDK initializes with the key
