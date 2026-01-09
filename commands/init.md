---
name: init
description: Interactive wizard for setting up a new Expo project with EAS, environments, and best practices
model: sonnet
skills:
  - expo-config
  - expo-eas
---

# Expo Project Initialisation Wizard

Guide the user through setting up a new or existing Expo project with proper EAS configuration, environment-specific bundle IDs, and best practices.

## Workflow Overview

1. **Detect project state** - New project or existing?
2. **Gather app information** - Name, bundle ID base, etc.
3. **Configure EAS** - Initialise and set up build profiles
4. **Set up environments** - Dev, preview, production bundle IDs
5. **Configure push notifications** - If needed
6. **Customise project** - README, CLAUDE.md updates
7. **Summary and next steps**

## Project Detection

First, check the current project state:

```bash
# Check if in an Expo project
ls app.json app.config.js package.json 2>/dev/null | head -5

# Check if EAS is already configured
ls eas.json 2>/dev/null && echo "EAS configured" || echo "EAS not configured"

# Check for existing Expo installation
grep -q "expo" package.json 2>/dev/null && echo "Expo found" || echo "Expo not found"
```

## Interactive Setup

### Step 1: Project Status

```typescript
AskUserQuestion({
  questions: [
    {
      question: "What's the current state of your project?",
      header: "Project",
      options: [
        {
          label: "New Expo project",
          description: "Just created with 'npx create-expo-app' or similar",
        },
        {
          label: "Existing Expo project",
          description: "Already have an Expo app, adding EAS",
        },
        {
          label: "Converting React Native project",
          description: "Existing RN app, adding Expo support",
        },
      ],
      multiSelect: false,
    },
  ],
});
```

### Step 2: App Information

```typescript
AskUserQuestion({
  questions: [
    {
      question: "What's your app name? (used for display)",
      header: "App Name",
      // Free text input via "Other"
      options: [
        {
          label: "Use existing name",
          description: "Keep the current app name from config",
        },
      ],
      multiSelect: false,
    },
    {
      question: "What's your bundle ID base? (e.g., com.yourcompany.yourapp)",
      header: "Bundle ID",
      options: [
        {
          label: "Use existing bundle ID",
          description: "Keep current bundle identifier",
        },
      ],
      multiSelect: false,
    },
    {
      question: "Will your app use push notifications?",
      header: "Push",
      options: [
        {
          label: "Yes",
          description: "Configure push notification credentials",
        },
        {
          label: "No",
          description: "Skip push notification setup",
        },
        {
          label: "Later",
          description: "I'll set this up later",
        },
      ],
      multiSelect: false,
    },
  ],
});
```

## Configuration Steps

### Step 3: EAS Initialisation

**If EAS not configured:**

```bash
# Install EAS CLI globally (if not installed)
which eas || npm install -g eas-cli

# Initialise EAS for the project
eas init
```

**Expected output:**
- Creates/updates `app.json` or `app.config.js` with EAS project ID
- Links project to EAS account

### Step 4: Create/Update app.config.js

Convert to dynamic config if using app.json, or update existing:

**Template for new app.config.js:**

```javascript
const IS_DEV = process.env.APP_VARIANT === 'development';
const IS_PREVIEW = process.env.APP_VARIANT === 'preview';

const getUniqueIdentifier = () => {
  if (IS_DEV) return '{BUNDLE_ID}.dev';
  if (IS_PREVIEW) return '{BUNDLE_ID}.preview';
  return '{BUNDLE_ID}';
};

const getAppName = () => {
  if (IS_DEV) return '{APP_NAME} (Dev)';
  if (IS_PREVIEW) return '{APP_NAME} (Preview)';
  return '{APP_NAME}';
};

export default {
  name: getAppName(),
  slug: '{SLUG}',
  version: '1.0.0',
  orientation: 'portrait',
  icon: './assets/icon.png',
  userInterfaceStyle: 'automatic',
  splash: {
    image: './assets/splash.png',
    resizeMode: 'contain',
    backgroundColor: '#ffffff',
  },
  assetBundlePatterns: ['**/*'],
  ios: {
    supportsTablet: false,
    bundleIdentifier: getUniqueIdentifier(),
    buildNumber: '1',
  },
  android: {
    adaptiveIcon: {
      foregroundImage: './assets/adaptive-icon.png',
      backgroundColor: '#ffffff',
    },
    package: getUniqueIdentifier(),
    versionCode: 1,
  },
  web: {
    favicon: './assets/favicon.png',
    bundler: 'metro',
  },
  plugins: [
    // Add plugins as needed
  ],
  extra: {
    eas: {
      projectId: '{PROJECT_ID}',
    },
  },
  updates: {
    url: 'https://u.expo.dev/{PROJECT_ID}',
  },
  runtimeVersion: {
    policy: 'appVersion',
  },
  owner: '{OWNER}',
};
```

**Replace placeholders:**
- `{BUNDLE_ID}` - Base bundle ID (e.g., com.yourcompany.yourapp)
- `{APP_NAME}` - Display name
- `{SLUG}` - URL-friendly name
- `{PROJECT_ID}` - EAS project ID (from eas init)
- `{OWNER}` - Expo account username

### Step 5: Create/Update eas.json

**Template for eas.json:**

```json
{
  "cli": {
    "version": ">= 16.18.0",
    "appVersionSource": "remote"
  },
  "build": {
    "base": {
      "env": {
        "APP_VARIANT": "production"
      }
    },
    "development": {
      "extends": "base",
      "developmentClient": true,
      "distribution": "internal",
      "env": {
        "APP_VARIANT": "development"
      },
      "ios": {
        "simulator": false
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleDebug"
      }
    },
    "development-simulator": {
      "extends": "development",
      "ios": {
        "simulator": true
      }
    },
    "preview": {
      "extends": "base",
      "distribution": "internal",
      "env": {
        "APP_VARIANT": "preview"
      },
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "extends": "base",
      "autoIncrement": true,
      "env": {
        "APP_VARIANT": "production"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "{APPLE_ID}",
        "ascAppId": "{ASC_APP_ID}"
      },
      "android": {
        "serviceAccountKeyPath": "./play-store-credentials.json",
        "track": "internal"
      }
    }
  }
}
```

### Step 6: Push Notification Setup (if selected)

**iOS Push Notifications:**

```bash
# Set up iOS push notification key
eas credentials --platform ios
# Select: Push Notifications: Manage your Apple Push Notifications Key
# Follow prompts to create or upload key
```

**Android Push Notifications (FCM):**

```bash
# Set up FCM key
eas credentials --platform android
# Select: FCM API Key: Manage your Firebase Cloud Messaging key
# Provide FCM server key from Firebase Console
```

**Add expo-notifications plugin:**

```bash
npx expo install expo-notifications
```

Update app.config.js plugins:
```javascript
plugins: [
  [
    'expo-notifications',
    {
      icon: './assets/notification-icon.png',
      color: '#ffffff',
    },
  ],
],
```

### Step 7: Update Project Documentation

**Create/Update README.md:**

```markdown
# {APP_NAME}

A React Native Expo app.

## Development

### Prerequisites

- Node.js 18+
- Expo CLI: `npm install -g expo-cli`
- EAS CLI: `npm install -g eas-cli`
- For iOS: Xcode (Mac only)
- For Android: Android Studio

### Setup

```bash
# Install dependencies
npm install

# Start development server
npx expo start
```

### Building

```bash
# Development build (iOS)
eas build --platform ios --profile development

# Development build (Android)
eas build --platform android --profile development

# Preview build (internal testing)
eas build --platform all --profile preview

# Production build
eas build --platform all --profile production --auto-submit
```

### Environment Variants

| Profile | Bundle ID | App Name |
|---------|-----------|----------|
| development | {BUNDLE_ID}.dev | {APP_NAME} (Dev) |
| preview | {BUNDLE_ID}.preview | {APP_NAME} (Preview) |
| production | {BUNDLE_ID} | {APP_NAME} |

## Project Structure

```
├── app/                 # App screens (if using Expo Router)
├── assets/              # Images, fonts, etc.
├── components/          # Reusable components
├── app.config.js        # Expo configuration
├── eas.json             # EAS Build configuration
└── package.json
```
```

**Create/Update .claude/CLAUDE.md (project-specific instructions):**

```markdown
# {APP_NAME} - Claude Code Instructions

## Project Overview

This is a React Native Expo app using EAS for builds and submissions.

## Key Configuration Files

- `app.config.js` - Expo configuration with dynamic bundle IDs
- `eas.json` - EAS Build profiles (development, preview, production)

## Environment Variants

The app uses `APP_VARIANT` environment variable for different builds:
- `development` - Dev client with `.dev` bundle ID suffix
- `preview` - Internal testing with `.preview` suffix
- `production` - Production release with base bundle ID

## Common Tasks

### Building
Run `/expo-toolkit:build` to prepare and validate before building.

### App Store Submission
- iOS: Run `/expo-toolkit:ios-preflight` before submitting
- Android: Run `/expo-toolkit:android-preflight` before submitting

### OTA Updates
Run `/expo-toolkit:update` to prepare OTA updates via EAS Update.

## Important Notes

- Bundle ID base: {BUNDLE_ID}
- EAS Project ID: {PROJECT_ID}
- Always use the appropriate build profile for the target environment
```

## Output Summary

After completing setup, display summary:

```
============================================
🎉 EXPO PROJECT INITIALISED
============================================

App Name: {APP_NAME}
Bundle ID Base: {BUNDLE_ID}
EAS Project: {PROJECT_ID}

📁 FILES CREATED/UPDATED
------------------------------------------
✅ app.config.js - Dynamic configuration with environment variants
✅ eas.json - Build profiles (development, preview, production)
✅ README.md - Project documentation
✅ .claude/CLAUDE.md - Claude Code instructions
{if push}
✅ Push notifications configured
{/if}

🏗️  BUILD PROFILES
------------------------------------------
| Profile     | Bundle ID                  | Use Case           |
|-------------|----------------------------|-------------------|
| development | {BUNDLE_ID}.dev            | Local development |
| preview     | {BUNDLE_ID}.preview        | Internal testing  |
| production  | {BUNDLE_ID}                | App store release |

📋 NEXT STEPS
------------------------------------------
1. Review app.config.js and customise as needed
2. Run `npx expo doctor` to verify project health
3. Create your first development build:

   eas build --platform ios --profile development

4. Set up submission credentials when ready:
   - iOS: Add appleId and ascAppId to eas.json submit section
   - Android: Create service account and add JSON key path

5. Before app store submission, run:
   - /expo-toolkit:ios-preflight
   - /expo-toolkit:android-preflight

============================================
USEFUL COMMANDS
============================================

# Start development server
npx expo start

# Build for development
eas build --platform ios --profile development

# Build for internal testing
eas build --platform all --profile preview

# Build for production with auto-submit
eas build --platform all --profile production --auto-submit

# Check project health
npx expo doctor

# Manage credentials
eas credentials

============================================
```

## Error Handling

### EAS Init Fails

```
🚫 EAS INITIALISATION FAILED
------------------------------------------
Error: {error message}

Common causes:
1. Not logged in to Expo: Run `eas login`
2. No Expo account: Create at https://expo.dev
3. Network issues: Check internet connection

After fixing, re-run this init command.
```

### Missing Dependencies

```
⚠️  MISSING DEPENDENCIES
------------------------------------------
The following packages are recommended:

  npx expo install expo-dev-client expo-updates

Install them? (Will run the command for you)
```

### Existing Configuration Conflict

```
⚠️  EXISTING CONFIGURATION DETECTED
------------------------------------------
Found existing:
- eas.json
- app.config.js

Options:
1. Merge with existing (recommended)
2. Replace existing (backup will be created)
3. Skip configuration files

What would you like to do?
```

## Handling Different Project States

### New Expo Project

Standard flow as described above.

### Existing Expo Project

1. Preserve existing configuration
2. Add missing EAS configuration
3. Suggest converting app.json to app.config.js if not done
4. Add environment variant support

### Converting React Native Project

1. Check for Expo compatibility
2. Install expo package: `npx install-expo-modules`
3. Follow standard EAS setup
4. May require `npx expo prebuild` for native code

## Files Modified/Created

| File | Action | Purpose |
|------|--------|---------|
| `app.config.js` | Create/Update | Dynamic Expo configuration |
| `eas.json` | Create/Update | EAS Build profiles |
| `README.md` | Update | Project documentation |
| `.claude/CLAUDE.md` | Create | Claude Code instructions |
| `package.json` | Update | Add expo-dev-client if needed |
