---
name: screenshots
description: Generate app store screenshots using Fastlane automation
argument-hint: "[ios|android|both]"
model: sonnet
skills:
  - fastlane
---

# App Store Screenshot Generation

Guide the user through setting up and running Fastlane for automated screenshot generation, or provide manual alternatives.

## Workflow Overview

1. **Detect current setup** - Check for existing Fastlane configuration
2. **Gather requirements** - Platforms, device sizes, locales
3. **Setup Fastlane** - If not configured
4. **Generate screenshots** - Run automation or guide manual capture
5. **Organise output** - Prepare for store upload

## Required Screenshot Sizes

### iOS (App Store Connect)

| Device | Size | Required |
|--------|------|----------|
| iPhone 6.7" | 1290 x 2796 | Yes (iPhone 15 Pro Max) |
| iPhone 6.5" | 1284 x 2778 | Yes (iPhone 14 Plus) |
| iPhone 5.5" | 1242 x 2208 | Yes (iPhone 8 Plus) |
| iPad Pro 12.9" | 2048 x 2732 | If iPad support |
| iPad Pro 12.9" (6th gen) | 2048 x 2732 | If iPad support |

**Note:** You can use one set of 6.7" screenshots for all iPhone sizes (scaled automatically).

### Android (Play Console)

| Type | Size | Required |
|------|------|----------|
| Phone | Min 320px, 16:9 or 9:16 | Yes (2-8 per listing) |
| 7" Tablet | 1024 x 500 min | If tablet support |
| 10" Tablet | 1024 x 500 min | If tablet support |

**Recommended:** 1080 x 1920 or 1440 x 2560 for phones.

## Interactive Setup

```typescript
AskUserQuestion({
  questions: [
    {
      question: "Which platform do you need screenshots for?",
      header: "Platform",
      options: [
        {
          label: "iOS",
          description: "App Store Connect screenshots",
        },
        {
          label: "Android",
          description: "Google Play Console screenshots",
        },
        {
          label: "Both",
          description: "Generate for both platforms",
        },
      ],
      multiSelect: false,
    },
    {
      question: "How would you like to capture screenshots?",
      header: "Method",
      options: [
        {
          label: "Fastlane automation (Recommended)",
          description: "Automated capture across devices and locales",
        },
        {
          label: "Manual capture with guidance",
          description: "I'll capture manually, provide device list",
        },
        {
          label: "Simulator screenshots",
          description: "Quick manual capture from simulators",
        },
      ],
      multiSelect: false,
    },
    {
      question: "Do you need localised screenshots?",
      header: "Locales",
      options: [
        {
          label: "English only",
          description: "Single language",
        },
        {
          label: "Multiple languages",
          description: "Different screenshots per locale",
        },
      ],
      multiSelect: false,
    },
  ],
});
```

## Fastlane Setup Check

```bash
# Check if Fastlane is installed
which fastlane || echo "Fastlane not installed"

# Check for existing Fastlane config
ls fastlane/Fastfile fastlane/Snapfile 2>/dev/null || echo "No Fastlane config"

# Check for iOS project
ls ios/*.xcworkspace ios/*.xcodeproj 2>/dev/null | head -1
```

## Fastlane Installation

If Fastlane not installed:

```bash
# Install via Homebrew (recommended)
brew install fastlane

# Or via RubyGems
sudo gem install fastlane

# Verify installation
fastlane --version
```

## iOS Screenshot Setup (Fastlane Snapshot)

### Initialise Snapshot

```bash
cd ios
fastlane snapshot init
```

This creates:
- `fastlane/Snapfile` - Configuration
- `fastlane/SnapshotHelper.swift` - UI test helper

### Configure Snapfile

```ruby
# fastlane/Snapfile

# Devices to capture
devices([
  "iPhone 15 Pro Max",   # 6.7"
  "iPhone 14 Plus",      # 6.5"
  "iPhone 8 Plus",       # 5.5"
  # "iPad Pro (12.9-inch) (6th generation)"  # If supporting iPad
])

# Languages to capture
languages([
  "en-GB",
  # "de-DE",
  # "fr-FR",
  # "ja",
])

# Where to save screenshots
output_directory("./fastlane/screenshots")

# Clear previous screenshots
clear_previous_screenshots(true)

# Scheme to use
scheme("YourAppUITests")

# Project/workspace
# workspace("YourApp.xcworkspace")
# project("YourApp.xcodeproj")

# Skip building (if already built)
# skip_open_summary(true)

# Override status bar
override_status_bar(true)
```

### Create UI Tests for Screenshots

Create `YourAppUITests/SnapshotTests.swift`:

```swift
import XCTest

class SnapshotTests: XCTestCase {

    override func setUpWithError() throws {
        continueAfterFailure = false
        let app = XCUIApplication()
        setupSnapshot(app)
        app.launch()
    }

    func testScreenshots() throws {
        let app = XCUIApplication()

        // Home screen
        snapshot("01_Home")

        // Navigate to feature
        app.buttons["Feature"].tap()
        snapshot("02_Feature")

        // Navigate to settings
        app.tabBars.buttons["Settings"].tap()
        snapshot("03_Settings")

        // Add more screens as needed
    }
}
```

### Run Snapshot

```bash
cd ios
fastlane snapshot

# Or with specific devices
fastlane snapshot --devices "iPhone 15 Pro Max"

# Or specific language
fastlane snapshot --languages "en-GB"
```

## Android Screenshot Setup (Fastlane Screengrab)

### Initialise Screengrab

```bash
cd android
fastlane screengrab init
```

This creates:
- `fastlane/Screengrabfile` - Configuration

### Configure Screengrabfile

```ruby
# fastlane/Screengrabfile

# Android package name
app_package_name("com.yourcompany.yourapp")

# Test package
tests_apk_path("app/build/outputs/apk/androidTest/debug/app-debug-androidTest.apk")

# App APK
app_apk_path("app/build/outputs/apk/debug/app-debug.apk")

# Locales to capture
locales(['en-US', 'en-GB'])

# Clear previous screenshots
clear_previous_screenshots(true)

# Output directory
output_directory("./fastlane/screenshots")

# End clean
ending_locale('en-US')
```

### Create Instrumented Tests

Create `app/src/androidTest/java/.../ScreenshotTests.kt`:

```kotlin
package com.yourcompany.yourapp

import androidx.test.ext.junit.rules.ActivityScenarioRule
import androidx.test.ext.junit.runners.AndroidJUnit4
import org.junit.Rule
import org.junit.Test
import org.junit.runner.RunWith
import tools.fastlane.screengrab.Screengrab
import tools.fastlane.screengrab.UiAutomatorScreenshotStrategy

@RunWith(AndroidJUnit4::class)
class ScreenshotTests {

    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)

    @Test
    fun captureScreenshots() {
        Screengrab.setDefaultScreenshotStrategy(UiAutomatorScreenshotStrategy())

        // Home screen
        Screengrab.screenshot("01_Home")

        // Navigate and capture more screens
        // ...
    }
}
```

### Add Screengrab Dependency

In `app/build.gradle`:

```groovy
androidTestImplementation 'tools.fastlane:screengrab:2.1.1'
```

### Run Screengrab

```bash
cd android
fastlane screengrab

# First build the APKs
./gradlew assembleDebug assembleAndroidTest

# Then run screengrab
fastlane screengrab
```

## Manual Screenshot Guide

If not using Fastlane automation:

### iOS Simulator Screenshots

```bash
# List available simulators
xcrun simctl list devices

# Boot a simulator
xcrun simctl boot "iPhone 15 Pro Max"

# Open Simulator app
open -a Simulator

# Take screenshot (Cmd+S in Simulator, or):
xcrun simctl io booted screenshot screenshot.png

# Screenshot with specific device
xcrun simctl io "iPhone 15 Pro Max" screenshot ~/Desktop/screenshot.png
```

**Required Simulators:**
```
============================================
iOS SCREENSHOT DEVICES
============================================
Run these simulators and capture screenshots:

1. iPhone 15 Pro Max (6.7")
   xcrun simctl boot "iPhone 15 Pro Max"

2. iPhone 14 Plus (6.5") - or use 6.7" screenshots
   xcrun simctl boot "iPhone 14 Plus"

3. iPhone 8 Plus (5.5")
   xcrun simctl boot "iPhone 8 Plus"

{if iPad support}
4. iPad Pro 12.9" (6th generation)
   xcrun simctl boot "iPad Pro (12.9-inch) (6th generation)"
{/if}

📋 CAPTURE CHECKLIST
------------------------------------------
For each device, capture:
□ Home/main screen
□ Key feature 1
□ Key feature 2
□ Key feature 3
□ Settings or profile

Tip: Use Cmd+S in Simulator to save screenshots
```

### Android Emulator Screenshots

```bash
# List available emulators
emulator -list-avds

# Start emulator
emulator -avd Pixel_6_Pro_API_34

# Take screenshot via adb
adb exec-out screencap -p > screenshot.png

# Or use Android Studio's screenshot button
```

**Required Emulators:**
```
============================================
ANDROID SCREENSHOT DEVICES
============================================
Run these emulators and capture screenshots:

1. Phone (1080x1920 or 1440x2560)
   - Pixel 6 Pro or similar

{if tablet support}
2. 7" Tablet (1200x1920)
   - Nexus 7 or similar

3. 10" Tablet (1600x2560)
   - Pixel Tablet or similar
{/if}

📋 CAPTURE CHECKLIST
------------------------------------------
For each device, capture:
□ Home/main screen
□ Key feature 1
□ Key feature 2
□ Key feature 3

Tip: Use adb or Android Studio to capture
```

## Screenshot Organisation

### Directory Structure

```
fastlane/screenshots/
├── en-GB/
│   ├── iPhone 15 Pro Max-01_Home.png
│   ├── iPhone 15 Pro Max-02_Feature.png
│   ├── iPhone 14 Plus-01_Home.png
│   └── ...
├── de-DE/
│   └── ...
└── android/
    ├── phoneScreenshots/
    │   ├── 01_Home.png
    │   └── ...
    └── sevenInchScreenshots/
        └── ...
```

### Renaming for Upload

App Store Connect expects specific naming or manual upload.

Play Console expects:
- `phoneScreenshots/` - Phone images
- `sevenInchScreenshots/` - 7" tablet images
- `tenInchScreenshots/` - 10" tablet images

## Framing Screenshots (Optional)

Add device frames and marketing text:

### Using Fastlane Frameit

```bash
# Install frameit
fastlane frameit

# Run on screenshots
fastlane frameit
```

Configure `frameit.json` for text overlays:

```json
{
  "default": {
    "title": {
      "font": "./fonts/MyFont.ttf",
      "color": "#000000"
    },
    "background": "#FFFFFF"
  },
  "data": [
    {
      "filter": "01_Home",
      "title": {
        "text": "Manage your tasks"
      }
    },
    {
      "filter": "02_Feature",
      "title": {
        "text": "Stay organised"
      }
    }
  ]
}
```

### Alternative: Use Design Tools

- Figma templates
- Sketch templates
- Online tools (AppMockUp, MockuPhone)

## Output Report

```
============================================
SCREENSHOT GENERATION COMPLETE
============================================
Platform: {platform}
Method: {method}
Date: {date}

📱 iOS SCREENSHOTS
------------------------------------------
Location: fastlane/screenshots/en-GB/

Device               Count    Status
iPhone 15 Pro Max    5        ✅ Complete
iPhone 14 Plus       5        ✅ Complete
iPhone 8 Plus        5        ✅ Complete
{if iPad}
iPad Pro 12.9"       5        ✅ Complete
{/if}

🤖 ANDROID SCREENSHOTS
------------------------------------------
Location: fastlane/screenshots/android/

Type                 Count    Status
Phone                5        ✅ Complete
{if tablet}
7" Tablet            5        ✅ Complete
10" Tablet           5        ✅ Complete
{/if}

📋 NEXT STEPS
------------------------------------------
1. Review screenshots in the output directory
2. (Optional) Add frames with `fastlane frameit`
3. Upload to App Store Connect / Play Console

iOS Upload:
  App Store Connect > App > Version > Screenshots

Android Upload:
  Play Console > Store presence > Main store listing > Graphics

============================================
```

## Troubleshooting

### Snapshot Not Finding Tests

```
🚫 UI TESTS NOT FOUND
------------------------------------------
Fastlane snapshot couldn't find UI tests.

Ensure:
1. UI Test target exists in Xcode
2. Test scheme is correct in Snapfile
3. SnapshotHelper.swift is added to UI Test target
4. setupSnapshot(app) called in setUp()
```

### Screengrab Build Failures

```
🚫 ANDROID BUILD FAILED
------------------------------------------
Screengrab couldn't build the test APK.

Try:
1. Build manually first:
   ./gradlew assembleDebug assembleAndroidTest

2. Check Screengrabfile paths are correct
3. Ensure screengrab dependency is added
```

### Simulator/Emulator Issues

```
🚫 DEVICE NOT AVAILABLE
------------------------------------------
The specified device is not available.

iOS: Check available devices with:
  xcrun simctl list devices

Android: Check available emulators with:
  emulator -list-avds

Install missing simulators via Xcode or Android Studio.
```

## Quick Reference

```bash
# iOS Fastlane Snapshot
cd ios
fastlane snapshot init           # First time setup
fastlane snapshot                # Run screenshots
fastlane snapshot --devices "iPhone 15 Pro Max"

# Android Fastlane Screengrab
cd android
fastlane screengrab init         # First time setup
./gradlew assembleDebug assembleAndroidTest
fastlane screengrab              # Run screenshots

# Manual iOS (Simulator)
xcrun simctl list devices
xcrun simctl boot "iPhone 15 Pro Max"
open -a Simulator
# Press Cmd+S to screenshot

# Manual Android (Emulator)
emulator -list-avds
emulator -avd Pixel_6_Pro_API_34
adb exec-out screencap -p > screenshot.png

# Frame screenshots
fastlane frameit
```

## Best Practices

1. **Show real content** - Use realistic data, not Lorem ipsum
2. **Highlight features** - Each screenshot should show a key feature
3. **Consistent style** - Same visual treatment across all screenshots
4. **Localise properly** - Show localised content if available
5. **Follow guidelines** - Check Apple/Google guidelines for requirements
6. **Test on real devices** - Verify screenshots look correct
7. **Update regularly** - Keep screenshots current with app changes
