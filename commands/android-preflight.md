---
name: android-preflight
description: Pre-submission checklist for Google Play Console - catches common rejection reasons
argument-hint: "[package_name or play_console_url]"
model: haiku
skills:
  - android-submission
  - revenuecat
---

# Google Play Console Pre-flight Check

Validate Google Play Console configuration before submission to catch common issues and policy violations.

## Setup Questionnaire

Before running checks, gather information about the user's setup using AskUserQuestion:

```typescript
AskUserQuestion({
  questions: [
    {
      question: "What is your app's monetisation model?",
      header: "Monetisation",
      options: [
        {
          label: "Freemium (IAP/Subscriptions)",
          description: "Free download with in-app purchases or subscriptions",
        },
        {
          label: "Paid Upfront",
          description: "One-time purchase price to download the app",
        },
        {
          label: "Free (No IAP)",
          description: "Completely free with no monetisation",
        },
      ],
      multiSelect: false,
    },
    {
      question: "Is this your first submission to Google Play?",
      header: "First Release",
      options: [
        {
          label: "Yes, first time",
          description: "App has never been published to Play Store",
        },
        {
          label: "No, updating existing app",
          description: "App already exists on Play Store",
        },
      ],
      multiSelect: false,
    },
    {
      question: "What testing track will you use?",
      header: "Testing Track",
      options: [
        {
          label: "Internal testing",
          description: "Up to 100 testers, instant availability",
        },
        {
          label: "Closed testing",
          description: "Limited testers, requires review",
        },
        {
          label: "Open testing",
          description: "Anyone can join, requires review",
        },
        {
          label: "Production",
          description: "Public release to all users",
        },
      ],
      multiSelect: false,
    },
    {
      question: "Do you want me to use browser automation to check Play Console?",
      header: "Browser",
      options: [
        {
          label: "Yes, use browser (Recommended)",
          description: "I'll navigate Play Console and verify each item",
        },
        {
          label: "No, walk me through manually",
          description: "I'll provide a checklist for you to verify",
        },
      ],
      multiSelect: false,
    },
  ],
});
```

### Conditional Checks Based on Setup

| Monetisation  | Skip These Checks                                     |
| ------------- | ----------------------------------------------------- |
| Free (No IAP) | IAP validation, RevenueCat cross-check, Subscription checks |
| Paid Upfront  | IAP validation, RevenueCat cross-check, Subscription checks |
| Freemium      | Run all checks                                        |

| First Release | Additional Checks                                     |
| ------------- | ----------------------------------------------------- |
| Yes           | Verify manual AAB upload guidance, 14-day testing requirement (personal accounts) |
| No            | Check version code increment, existing listing updates |

## Prerequisites Check

**If browser automation selected:**

```typescript
// Test Claude in Chrome MCP connectivity
mcp__claude-in-chrome__tabs_context_mcp({ createIfEmpty: true });
```

If this fails, **STOP IMMEDIATELY** and display:

```
🚨 CLAUDE IN CHROME MCP NOT AVAILABLE

The android-preflight command requires browser automation via Claude in Chrome.

To fix:
1. Ensure Chrome is running
2. Verify Claude in Chrome extension is installed and enabled
3. Check MCP server is connected in Claude Code settings

Cannot proceed without browser access. Re-run with manual walkthrough option.
```

## Authentication Check

After navigating to Play Console, verify the user is logged in:

1. Navigate to Play Console
2. Take a screenshot
3. Check if the page shows app content (not a login screen)

If login screen detected, **STOP** and display:

```
🔐 GOOGLE PLAY CONSOLE LOGIN REQUIRED

Please log in to Google Play Console in your browser, then run this command again.

URL: https://play.google.com/console
```

## Input Handling

Accept either:
- Play Console URL: `https://play.google.com/console/u/0/developers/.../app/.../...`
- Package name: `com.yourcompany.yourapp`

Extract package name and construct base URL if needed.

## Checklist Workflow

### 1. App Access & Dashboard

Navigate to: Play Console → Your app

**Basic Verification:**
- [ ] App exists in Play Console
- [ ] Correct package name displayed
- [ ] App status is not "Suspended" or "Removed"

### 2. Store Listing (Main Store Listing)

Navigate to: Grow → Store presence → Main store listing

**App Details:**
- [ ] App name present (≤30 characters on Play Store)
- [ ] Short description present (≤80 characters)
- [ ] Full description present (≤4000 characters)
- [ ] No placeholder text (Lorem ipsum, TBD, etc.)
- [ ] No competitor names or trademarked terms
- [ ] No pricing information in description (use in-app purchases section)

**Graphics:**
- [ ] App icon uploaded (512x512 PNG)
- [ ] Feature graphic uploaded (1024x500)
- [ ] At least 2 phone screenshots uploaded
- [ ] Screenshots show actual app functionality (not splash screens)
- [ ] If tablet support: Tablet screenshots uploaded

**Categorisation:**
- [ ] App category selected
- [ ] Category appropriate for app content

### 3. Store Settings

Navigate to: Grow → Store presence → Store settings

**Contact Details:**
- [ ] Developer email provided
- [ ] Privacy policy URL configured
- [ ] Privacy policy URL is accessible (test with WebFetch)

### 4. Content Rating

Navigate to: Policy and programs → App content → Content rating

**Rating Questionnaire:**
- [ ] Content rating questionnaire completed
- [ ] Rating not "Unrated" (will cause rejection)
- [ ] Rating appropriate for app content

### 5. Target Audience and Content

Navigate to: Policy and programs → App content → Target audience and content

**Target Age:**
- [ ] Target age group selected
- [ ] If targeting children: Additional requirements met
- [ ] Content appropriate for declared age group

### 6. Privacy Policy

Navigate to: Policy and programs → App content → Privacy policy

**Privacy Configuration:**
- [ ] Privacy policy URL provided
- [ ] Privacy policy accessible from within app (both required)
- [ ] Privacy policy URL returns 200 OK (test with WebFetch)

### 7. Data Safety

Navigate to: Policy and programs → App content → Data safety

**Data Safety Form:**
- [ ] Data safety section completed (not "Start")
- [ ] Data collection types declared accurately
- [ ] Data sharing disclosed if applicable
- [ ] Security practices described

### 8. Ads Declaration

Navigate to: Policy and programs → App content → Ads

**Ads Configuration:**
- [ ] Ads declaration completed
- [ ] If app contains ads: Declared as "Yes"
- [ ] If no ads: Declared as "No"

### 9. App Access (If Login Required)

Navigate to: Policy and programs → App content → App access

**Demo Credentials:**
- [ ] If app requires login: Access instructions provided
- [ ] Demo credentials work (test before submission)
- [ ] Demo account has access to all features being reviewed

### 10. In-App Products (If Freemium)

Navigate to: Monetize → Products → In-app products / Subscriptions

**Product Status:**
- [ ] Products exist in "Active" status
- [ ] Product IDs match what's configured in app/RevenueCat
- [ ] Prices configured for target countries
- [ ] Product names and descriptions appropriate

**Subscription-Specific:**
- [ ] Grace period configured appropriately
- [ ] Base plans defined
- [ ] Offers configured (if applicable)

### 11. RevenueCat Cross-Check (If Freemium)

**If RevenueCat MCP available:**
```typescript
const apps = mcp__revenuecat__mcp_RC_list_apps({ project_id: "proj..." });
const products = mcp__revenuecat__mcp_RC_list_products({ project_id: "proj..." });
```

**Cross-Validation:**
- [ ] Play Store app configured in RevenueCat
- [ ] Product IDs match between RC and Play Console
- [ ] Current offering is set
- [ ] Entitlements mapped to products

**If RevenueCat MCP not available:**
Navigate to RevenueCat dashboard via browser and verify:
- [ ] Android app exists with correct package name
- [ ] Products listed with matching IDs
- [ ] Offerings configured

### 12. Testing Track Setup

Navigate to: Release → Testing → [Internal/Closed/Open testing]

**For First Release (Personal Accounts):**
- [ ] Internal testing track available (recommended starting point)
- [ ] If going directly to production with personal account:
  - [ ] Closed testing with 12+ testers for 14+ days completed
  - [ ] OR organisation account (exempt from requirement)

**Track Configuration:**
- [ ] Testers list configured
- [ ] Release notes provided (for closed/open testing)

### 13. Production Readiness

Navigate to: Release → Production

**Release Requirements:**
- [ ] All setup tasks show green checkmarks
- [ ] No blocking issues in "Publishing overview"
- [ ] Countries/regions selected for distribution
- [ ] Managed publishing: Understand publish timing

## First Release Special Guidance

If this is the first submission:

```
📱 FIRST RELEASE REQUIREMENTS
------------------------------------------
For your first Android release:

1. AAB Upload: You must manually upload your first AAB file
   - Build: eas build --platform android --profile production
   - Download AAB from EAS dashboard
   - Upload to Play Console → Release → Production → Create release

2. After first upload, EAS --auto-submit will work for future releases

3. Personal Account Testing Requirement:
   - Personal developer accounts (not organisation accounts)
   - Must run closed testing with 12+ testers for 14+ days
   - Before production access is granted
   - This is Google's fraud prevention policy

4. Review Timeline:
   - First submission: 7+ days typical
   - Updates to existing apps: 1-3 days typical
```

## URL Validation

Test that all configured URLs are reachable:

```typescript
WebFetch({
  url: privacyPolicyUrl,
  prompt: "Does this page load successfully? Return YES or NO.",
});
```

URLs to validate:
- [ ] Privacy Policy URL - Must be HTTPS, must load
- [ ] Support URL (if set) - Must be HTTPS, must load

## Code Verification

Search codebase for compliance with Google Play requirements:

```typescript
// Check for restore purchases implementation
Grep({ pattern: "restorePurchases|restoreTransactions", type: "ts" });

// Check for privacy policy link in-app
Grep({ pattern: "privacy|privacyPolicy|privacy-policy", type: "ts" });

// Check for billing client
Grep({ pattern: "BillingClient|useIAP|react-native-iap|revenuecat", type: "ts" });
```

**Required in Code:**
- [ ] Privacy policy accessible from within app
- [ ] If IAP: Billing library properly integrated
- [ ] If subscriptions: Subscription terms displayed before purchase

## Output Format

Generate a structured report:

```
============================================
GOOGLE PLAY CONSOLE PRE-FLIGHT CHECK
============================================
App: {app_name}
Package: {package_name}
Track: {testing_track}
Date: {date}

🚫 BLOCKING ISSUES (will cause rejection)
------------------------------------------
- [Area]: [Specific issue]
- [Area]: [Specific issue]

⚠️  WARNINGS (may cause rejection or delays)
------------------------------------------
- [Area]: [Issue description]

✅ PASSED CHECKS
------------------------------------------
- Store listing complete
- Content rating configured
- Privacy policy accessible
- [etc.]

📋 MANUAL VERIFICATION NEEDED
------------------------------------------
- Demo account credentials work
- Screenshots accurately represent app
- Data safety responses accurate
- [etc.]

📱 FIRST RELEASE NOTES (if applicable)
------------------------------------------
- Remember: First AAB must be manually uploaded
- Testing requirement: [status]

============================================
RECOMMENDATION: [Ready to submit / Fix X issues first]
============================================
```

## Common Rejection Reasons

### Policy Violations

| Policy Area | Common Issue | How to Fix |
|-------------|--------------|------------|
| Deceptive behavior | Misleading app functionality | Ensure description matches actual features |
| User data | Privacy policy missing/inaccessible | Add accessible privacy policy |
| Malicious behavior | Requesting unnecessary permissions | Remove unused permissions |
| Impersonation | Using trademarked content | Remove trademarked material |
| Inappropriate content | Age rating mismatch | Re-submit content rating questionnaire |

### Technical Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Version code conflict | Same version already exists | Increment `versionCode` in build |
| Signing key mismatch | Different upload key | Use correct keystore |
| Target API level | API level too low | Update `targetSdkVersion` to meet requirements |
| 64-bit requirement | Missing arm64 support | Ensure build includes 64-bit libraries |

### Store Listing Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Missing graphics | Required assets not uploaded | Upload all required screenshots/graphics |
| Description violations | Prohibited content in text | Review and update description |
| Keyword stuffing | Excessive keywords | Use natural language in description |

## Navigation Paths

- Dashboard: `https://play.google.com/console/u/0/developers/{dev_id}/app/{app_id}`
- Store listing: `.../store-listing`
- Content rating: `.../content-rating`
- Data safety: `.../data-safety`
- In-app products: `.../managed-products`
- Subscriptions: `.../subscriptions`
- Testing: `.../tracks/{track_id}`
- Production: `.../production`

## Speed Optimisations

- Use `haiku` model for fast validation
- Take screenshots at key checkpoints only
- Don't read full page content - just validate presence/absence
- If manual walkthrough: Provide concise checklist for user to follow

## Manual Walkthrough Mode

If user selected manual walkthrough instead of browser automation, provide this checklist:

```
📋 GOOGLE PLAY CONSOLE MANUAL CHECKLIST
============================================

Open Play Console: https://play.google.com/console

Navigate to each section and verify:

□ STORE LISTING (Grow → Store presence → Main store listing)
  □ App name, short description, full description filled
  □ App icon (512x512) uploaded
  □ Feature graphic (1024x500) uploaded
  □ At least 2 phone screenshots uploaded
  □ Screenshots show actual app functionality

□ STORE SETTINGS (Grow → Store presence → Store settings)
  □ Developer email provided
  □ Privacy policy URL configured and accessible

□ CONTENT RATING (Policy and programs → App content → Content rating)
  □ Rating questionnaire completed (not "Unrated")

□ TARGET AUDIENCE (Policy and programs → App content → Target audience)
  □ Target age group selected

□ DATA SAFETY (Policy and programs → App content → Data safety)
  □ Data safety form completed

□ APP ACCESS (Policy and programs → App content → App access)
  □ If login required: Demo credentials provided

□ IN-APP PRODUCTS (If applicable - Monetize → Products)
  □ Products in "Active" status
  □ Product IDs match app code

□ RELEASE (Release → Production/Testing)
  □ All setup tasks show green checkmarks
  □ AAB uploaded (manual for first release)
  □ Release notes provided

Let me know which items pass or fail, and I'll help resolve any issues.
```

## Error Handling

If a section fails to load or is inaccessible:
- Note as "Unable to verify: [reason]"
- Continue with other checks
- Include in final report

If logged out:
- Stop immediately
- Report: "Authentication required - please log in to Play Console"
