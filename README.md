# Expo Toolkit

A comprehensive Claude Code plugin for React Native Expo app development - from project initialisation through app store submission and maintenance.

## Features

- **Project Initialisation** - Interactive wizard for setting up Expo + EAS with environment-specific bundle IDs
- **Build Preparation** - Pre-flight validation before running EAS builds
- **OTA Updates** - Safe deployment of EAS Updates with environment checks
- **App Store Submission** - Comprehensive checklists for iOS App Store Connect and Google Play Console
- **Screenshot Automation** - Fastlane setup for automated screenshot generation
- **Changelog Generation** - User-friendly "What's New" text from git history
- **Monetisation** - RevenueCat integration guidance and cross-platform product setup

## Requirements

- Claude Code CLI
- Claude in Chrome MCP (required for browser automation in preflight commands)
- RevenueCat MCP (optional, enhances IAP validation)
- Context7 MCP (optional, enhances documentation research)

## Installation

### Via Marketplace (Recommended)

```bash
# Add the marketplace
/plugin marketplace add rahulkeerthi/expo-toolkit

# Install the plugin
/plugin install expo-toolkit

# Verify installation
/help
```

### Manual Installation

Clone and add as a local plugin:

```bash
git clone https://github.com/rahulkeerthi/expo-toolkit.git
cd expo-toolkit
/plugin install --plugin-dir .
```

## Commands

### `/init` - Project Initialisation

Interactive wizard for setting up a new or existing Expo project with EAS configuration:

```bash
/init
```

Sets up:
- EAS initialisation and project linking
- Dynamic `app.config.js` with APP_VARIANT pattern
- Environment-specific bundle IDs (dev, preview, production)
- Build profiles in `eas.json`
- Push notification configuration (optional)
- Project documentation (README, CLAUDE.md)

### `/build` - Build Preparation

Validates configuration before running EAS builds:

```bash
/build
/build production --ios
/build preview --all
```

Validates:
- eas.json and app.config.js configuration
- iOS/Android credentials
- Environment variables and secrets
- Project health (expo doctor)
- Version numbers
- Submission configuration (if auto-submit)

Outputs the ready-to-run `eas build` command.

### `/update` - OTA Update Preparation

Prepares for EAS Update deployment:

```bash
/update
/update production --message "Bug fixes"
```

Validates:
- EAS Update configuration
- Runtime version compatibility
- Environment variables (no secrets leak)
- Bundle size
- Channel/branch setup

### `/ios-preflight` - iOS App Store Connect Check

Pre-submission checklist for App Store Connect:

```bash
/ios-preflight
/ios-preflight 1234567890
/ios-preflight https://appstoreconnect.apple.com/apps/...
```

Validates:
- Build uploaded and selected
- Screenshots for all device sizes
- App metadata and descriptions
- IAPs/subscriptions attached to version
- Privacy policy and labels
- Age rating
- Review information and demo credentials
- RevenueCat cross-check (if available)

### `/android-preflight` - Google Play Console Check

Pre-submission checklist for Play Console:

```bash
/android-preflight
/android-preflight com.yourcompany.yourapp
```

Validates:
- Store listing (name, descriptions, graphics)
- Content rating and target audience
- Data safety form
- Privacy policy
- In-app products and subscriptions
- Testing track configuration
- First release requirements (manual AAB upload)
- RevenueCat cross-check (if available)

### `/screenshots` - Screenshot Generation

Guide for Fastlane screenshot automation:

```bash
/screenshots
/screenshots ios
/screenshots android
```

Helps with:
- Fastlane setup and configuration
- Device/simulator setup for required sizes
- Localised screenshot capture
- Framing with device frames and text

### `/changelog` - Changelog Generation

Generate "What's New" text for app store submissions:

```bash
/changelog
/changelog 1.3.0
```

Features:
- Git history analysis
- PR-based change extraction
- User-friendly transformation
- Platform-specific formatting (iOS 4000 chars, Android 500 chars)

## Skills

The plugin includes expert knowledge in:

| Skill | Description |
|-------|-------------|
| `expo-eas` | EAS Build, Submit, credentials management, build profiles |
| `expo-config` | app.config.js, environment variants, APP_VARIANT pattern |
| `ios-submission` | App Store Connect, review guidelines, submission process |
| `android-submission` | Play Console, testing tracks, store listing requirements |
| `revenuecat` | SDK integration, entitlements, offerings, paywalls |
| `fastlane` | Screenshot automation, snapshot, screengrab, frameit |

## Agents

Specialised agents that can be triggered proactively:

| Agent | Description |
|-------|-------------|
| `expo-docs` | Research Expo/RN/EAS documentation after unexpected errors |
| `revenuecat-docs` | Research RevenueCat documentation for monetisation questions |
| `build-troubleshooter` | Diagnose and troubleshoot EAS build failures |

## Environment-Specific Bundle IDs

The plugin recommends the APP_VARIANT pattern for running multiple environments side-by-side:

| Profile | Bundle ID | App Name |
|---------|-----------|----------|
| development | com.company.app.dev | MyApp (Dev) |
| preview | com.company.app.preview | MyApp (Preview) |
| production | com.company.app | MyApp |

## First Release Notes

### iOS
- Auto-submit works immediately with proper credentials
- TestFlight distribution available after first build

### Android
- **First AAB must be manually uploaded** to Play Console
- After first upload, `--auto-submit` works for subsequent builds
- Personal accounts require 12+ testers for 14+ days before production access

## Example Workflow

```bash
# 1. Initialise new project
/init

# 2. Develop your app...

# 3. Prepare for first build
/build development --ios

# 4. After development, prepare production build
/build production --all

# 5. Before App Store submission
/ios-preflight

# 6. Before Play Store submission
/android-preflight

# 7. Generate changelog
/changelog

# 8. After store approval, deploy OTA updates
/update production --message "Bug fixes"
```

## Contributing

Contributions welcome! Please open an issue or PR.

## License

MIT License - see LICENSE file.

## Credits

Created by Rahul Keerthi + Claude

Based on:
- [Expo Documentation](https://docs.expo.dev)
- [EAS Documentation](https://docs.expo.dev/eas/)
- [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Google Play Policy](https://play.google.com/console/about/guides/releasewithconfidence/)
- [RevenueCat Documentation](https://docs.revenuecat.com)
