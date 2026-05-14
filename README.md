<div align="center">
🗺️ Google Maps API Key — Secure Setup Guide
Zero hardcoded keys. Zero secrets in Git. Production-ready.

</div>

⚠️ The Problem
Never hardcode your API key. Here's why every location is risky:
FileRiskandroid/app/src/main/AndroidManifest.xml🔴 Exposed after APK decompileandroid/app/build.gradle.kts🔴 Exposed in source codeios/Runner/Info.plist🔴 Exposed in IPA inspectionios/Runner/AppDelegate.swift🔴 Exposed in source code

🔐 The Secure Chain
ANDROID
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
.env (git-ignored) → build.gradle.kts → AndroidManifest.xml → Runtime

iOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Xcode Build Settings (never committed) → Info.plist → AppDelegate.swift → Runtime

📋 Table of Contents

Step 1 — .env File
Step 2 — Android: build.gradle.kts
Step 3 — Android: AndroidManifest.xml
Step 4 — iOS: Xcode Build Settings
Step 5 — iOS: Info.plist
Step 6 — iOS: AppDelegate.swift
Step 7 — CI/CD: GitHub Actions
Step 8 — Verify
Key Flow Reference


Step 1 — .env File (Project Root)
Create a .env file at your project root with this exact key name:
envGOOGLE_MAPS_SDK_KEY=AIzaSy..........................
Then make sure it's git-ignored:
gitignore# .gitignore
.env
android/local.properties
android/key.properties

🛡️ This file never leaves your machine.


Step 2 — Android: android/app/build.gradle.kts
Your build.gradle.kts already has the correct setup — no changes needed.
kotlin// Befor Section android {----} 
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

Step 3 — Android: AndroidManifest.xml
Replace the hardcoded key with the placeholder variable:
xml<!-- ❌ REMOVE this -->
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="AIzaSy.........................."/>

<!-- ✅ ADD this -->
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="${GOOGLE_MAPS_SDK_KEY}"/>

Step 4 — iOS: Add the Key in Xcode

This is the iOS equivalent of your .env file. The key lives only inside Xcode — never in any file pushed to Git.

Open your project
Open Xcode → Runner.xcworkspace
(Always open .xcworkspace, never .xcodeproj)
Add the key as a User-Defined Setting
Left sidebar
└── Click the Runner project (top-level blue icon)

Under TARGETS
└── Click Runner

Click the "Build Settings" tab

At the top of Build Settings
└── Select "All" and "Combined"

Click the "+" button (top-left of the Build Settings list)
└── Choose "Add User-Defined Setting"

A new row appears at the bottom under "User-Defined"
└── Type the name exactly: GOOGLE_MAPS_SDK_KEY
└── Press Enter

Click the arrow (▶) on the left of the row to expand it
└── Set the value for each configuration:
    Debug   → AIzaSy..........................
    Profile → AIzaSy..........................
    Release → AIzaSy..........................

Press Cmd+S to save

✅ Now $(GOOGLE_MAPS_SDK_KEY) in Info.plist automatically resolves at build time for all configurations.


Step 5 — iOS: Info.plist
Replace the hardcoded key:
xml<!-- ❌ REMOVE this -->
<key>GMSApiKey</key>
<string>AIzaSy..........................</string>

<!-- ✅ ADD this — resolves from Xcode Build Settings at build time -->
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

Step 7 — Verify
Android
bashflutter clean
flutter pub get
flutter run

🟡 If the map shows grey tiles → the key was not injected.
Check that the key name in .env is exactly GOOGLE_MAPS_SDK_KEY (case-sensitive).

iOS
bashflutter clean
flutter pub get
cd ios && pod install && cd ..
flutter run

🟡 If the app crashes on launch → verify GOOGLE_MAPS_SDK_KEY exists in Xcode Build Settings under User-Defined for all three configurations (Debug / Profile / Release).


🗺️ Complete Key Flow Reference
<details>
<summary><strong>Android Flow</strong></summary>
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
Runtime — Google Maps SDK initializes with the key ✅
</details>
<details>
<summary><strong>iOS Flow</strong></summary>
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
Runtime — Google Maps SDK initializes with the key ✅
</details>

<div align="center">
Made with ❤️ for Flutter developers who care about security
</div>
