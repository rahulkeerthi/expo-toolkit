# App Store Preflight Skill

A Claude Code skill that validates your App Store Connect configuration before submission, catching common rejection reasons.

Built to prevent rejections like:
- Guideline 2.1 - App Completeness (IAPs not submitted with version)
- Guideline 2.3 - Accurate Metadata (screenshots, descriptions, age ratings)
- Guideline 3.1.1 - In-App Purchase (restore purchases, loot box odds)
- Guideline 3.1.2 - Subscriptions (trial config, upgrade/downgrade)
- Guideline 5.1.1 - Data Collection (privacy labels, privacy policy)

## Requirements

- Claude Code CLI
- Claude in Chrome MCP (required for browser automation)
- RevenueCat MCP (optional, falls back to browser if unavailable)

## Installation

Copy the `.claude` directory into your project root:

```bash
cp -r .claude /path/to/your/project/
```

Or manually copy:
- `.claude/commands/app-store-preflight.md`
- `.claude/skills/app-store-preflight/SKILL.md`

## Usage

```bash
# Run with App Store Connect URL
/app-store-preflight https://appstoreconnect.apple.com/apps/1234567890/distribution/ios/version/inflight

# Or just the app ID
/app-store-preflight 1234567890
```

The skill will:

1. Ask about your app configuration (monetization, paywall, iPad support)
2. Detect available MCPs (Claude in Chrome, RevenueCat)
3. Navigate through App Store Connect sections
4. Validate against Apple's App Review Guidelines
5. Cross-check with RevenueCat if available
6. Search your codebase for compliance issues
7. Generate a structured report with blocking issues, warnings, and passed checks

## What It Checks

### App Store Connect

- Build uploaded and selected
- Screenshots for all device sizes (show app in use, not splash screens)
- App name ≤30 characters
- Description without trademarked terms
- What's New describes changes
- IAPs/subscriptions attached to version (common rejection cause!)
- Review screenshots for each product
- Contact information for App Review
- Demo credentials if login required
- Privacy policy URL
- Age rating
- Privacy labels

### RevenueCat Cross-Check

- iOS app configured with matching bundle ID
- Product IDs match between RC and App Store Connect
- Current offering set
- Entitlements have products attached

### Code Verification

- Restore purchases mechanism implemented
- Privacy policy accessible in-app (not just App Store)
- No alternative payment bypass (license keys, QR codes)
- Trial duration matches App Store Connect config

## Example Output

```
============================================
APP STORE CONNECT PRE-FLIGHT CHECK
============================================
App: MyApp
Version: 1.0.0
Date: 2025-01-07

BLOCKING ISSUES (will cause rejection)
------------------------------------------
- In-App Purchases: 2 subscriptions not attached to version
- App Review: Demo credentials missing (app requires login)

WARNINGS (may cause rejection or delays)
------------------------------------------
- Screenshots: iPad Pro 12.9" missing
- Metadata: Promotional text empty

PASSED CHECKS
------------------------------------------
- Version metadata complete
- Privacy policy configured
- Age rating set
- Build uploaded and selected

MANUAL VERIFICATION NEEDED
------------------------------------------
- Demo account credentials work
- Screenshots accurately represent app

============================================
RECOMMENDATION: Fix 2 blocking issues first
============================================
```

## Customization

The skill uses an interactive questionnaire to skip irrelevant checks:

| Monetization | Skipped Checks |
|-------------|----------------|
| Free (No IAP) | IAP validation, RevenueCat, Paywall |
| Paid Upfront | IAP validation, RevenueCat, Paywall |
| Freemium | None (runs all checks) |

| iPad Support | Screenshot Check |
|-------------|-----------------|
| iPhone Only | Skip iPad Pro validation |
| Universal | Require iPad Pro 12.9" |

## Contributing

Contributions welcome! Please open an issue or PR.

## License

MIT License - see LICENSE file.

## Credits

Created by [@rahulkeerthi](https://github.com/rahulkeerthi)

Based on Apple's App Review Guidelines:
- https://developer.apple.com/app-store/review/guidelines/
