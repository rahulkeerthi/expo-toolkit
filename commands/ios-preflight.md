---
name: ios-preflight
description: Pre-submission checklist for iOS App Store Connect - catches common rejection reasons
argument-hint: "[app_id or app_store_connect_url]"
model: haiku
skills:
  - ios-submission
  - revenuecat
---

# iOS App Store Connect Pre-flight Check

Validate App Store Connect configuration before submission to catch common rejection reasons.

## Setup Questionnaire

Before running checks, gather information about the user's setup using AskUserQuestion:

```typescript
AskUserQuestion({
  questions: [
    {
      question: "What is your app's monetization model?",
      header: "Monetization",
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
          description: "Completely free with no monetization",
        },
      ],
      multiSelect: false,
    },
    {
      question: "Which paywall implementation are you using?",
      header: "Paywall",
      options: [
        {
          label: "RevenueCat Paywalls",
          description: "Using RC's built-in paywall UI configured in dashboard",
        },
        {
          label: "Custom Implementation",
          description: "Custom paywall UI with RevenueCat SDK for purchases",
        },
        {
          label: "No Paywall",
          description: "No subscription/purchase UI (free app or paid upfront)",
        },
      ],
      multiSelect: false,
    },
    {
      question: "Is your app universal (supports iPad)?",
      header: "iPad",
      options: [
        {
          label: "iPhone Only",
          description: "App only runs on iPhone",
        },
        {
          label: "Universal (iPhone + iPad)",
          description: "App runs on both iPhone and iPad",
        },
      ],
      multiSelect: false,
    },
  ],
});
```

### Conditional Checks Based on Setup

Based on user responses, adjust the validation:

| Monetization  | Skip These Checks                                     |
| ------------- | ----------------------------------------------------- |
| Free (No IAP) | IAP validation, RevenueCat cross-check, Paywall check |
| Paid Upfront  | IAP validation, RevenueCat cross-check, Paywall check |
| Freemium      | Run all checks                                        |

| Paywall Type        | Check Approach                                   |
| ------------------- | ------------------------------------------------ |
| RevenueCat Paywalls | Validate RC dashboard paywall config via browser |
| Custom              | Grep codebase for implementation patterns        |
| No Paywall          | Skip paywall checks entirely                     |

| iPad Support | Screenshot Check                    |
| ------------ | ----------------------------------- |
| iPhone Only  | Skip iPad Pro screenshot validation |
| Universal    | Require iPad Pro 12.9" screenshots  |

### MCP Availability Detection

After setup questions, automatically detect available MCPs:

```typescript
// Step 1: Test Claude in Chrome MCP (REQUIRED)
const chromeResult = mcp__claude-in-chrome__tabs_context_mcp({ createIfEmpty: true });
const hasChromeInMCP = !chromeResult.error;

// Step 2: Test RevenueCat MCP (OPTIONAL - only if freemium)
// Note: The MCP tool name varies by installation. Common patterns:
// - mcp__revenuecat__mcp_RC_get_project()
// - mcp__revenuecat-{project}__mcp_RC_get_project()
let hasRevenueCatMCP = false;
if (monetization === "Freemium") {
  try {
    const rcResult = mcp__revenuecat__mcp_RC_get_project();
    hasRevenueCatMCP = !rcResult.error;
  } catch {
    hasRevenueCatMCP = false;
  }
}
```

Display detected configuration:

```
📋 CONFIGURATION DETECTED
------------------------------------------
Monetization: Freemium (IAP/Subscriptions)
Paywall: Custom Implementation
iPad Support: iPhone Only

MCP Status:
  ✓ Claude in Chrome: Available
  ✓ RevenueCat MCP: Available

Checks to run:
  ✓ Version metadata
  ✓ IAP/Subscription validation
  ✓ RevenueCat cross-check (via MCP)
  ✓ Custom paywall code analysis
  ✓ App Review information
  ✓ Privacy configuration
  ✓ URL validation
  ✗ iPad screenshots (iPhone only)
```

If Claude in Chrome MCP is not available, stop and show error (see Prerequisites Check below).

If RevenueCat MCP is not available but user has freemium app, fall back to browser automation for RC checks.

## Prerequisites Check

**CRITICAL: First verify Claude in Chrome MCP is available.**

```typescript
// Test Claude in Chrome MCP connectivity
mcp__claude-in-chrome__tabs_context_mcp({ createIfEmpty: true });
```

If this fails or returns an error, **STOP IMMEDIATELY** and display:

```
🚨 CLAUDE IN CHROME MCP NOT AVAILABLE

The app-store-preflight command requires browser automation via Claude in Chrome.

To fix:
1. Ensure Chrome is running
2. Verify Claude in Chrome extension is installed and enabled
3. Check MCP server is connected in Claude Code settings

Cannot proceed without browser access.
```

If successful, you should receive a response with `availableTabs` and a `tabGroupId`. Use a tab from this group or create a new one for navigation.

## Authentication Check

After navigating to App Store Connect, verify the user is logged in:

1. Navigate to the app's version page
2. Take a screenshot
3. Check if the page shows app content (not a login screen)

If login screen detected, **STOP** and display:

```
🔐 APP STORE CONNECT LOGIN REQUIRED

Please log in to App Store Connect in your browser, then run this command again.

URL: https://appstoreconnect.apple.com
```

Signs of login required:

- Page title contains "Sign In"
- URL redirects to `appleid.apple.com` or `idmsa.apple.com`
- Page shows Apple ID login form

## Input Handling

Accept either:

- App Store Connect URL: `https://appstoreconnect.apple.com/apps/XXXXXXXXXX/...`
- App ID: `XXXXXXXXXX` (10-digit number)

Extract app ID and construct base URL: `https://appstoreconnect.apple.com/apps/{app_id}`

## RevenueCat Cross-Check (Optional)

**Check if RevenueCat MCP is available:**

```typescript
// Test RevenueCat MCP connectivity
// Tool name varies - try common patterns
mcp__revenuecat__mcp_RC_get_project();
```

If RevenueCat MCP is available, perform cross-validation:

### Fetch RevenueCat Configuration

```typescript
// Get all apps, products, offerings, and entitlements
const apps = mcp__revenuecat__mcp_RC_list_apps({ project_id: "proj..." });
const products = mcp__revenuecat__mcp_RC_list_products({ project_id: "proj..." });
const offerings = mcp__revenuecat__mcp_RC_list_offerings({ project_id: "proj..." });
const entitlements = mcp__revenuecat__mcp_RC_list_entitlements({ project_id: "proj..." });
```

### Cross-Validation Checks

1. **App Store App Exists in RC**
   - [ ] iOS app configured with matching bundle ID
   - [ ] App type is `app_store`

2. **Product ID Matching**
   - [ ] Each RC product `store_identifier` exists in App Store Connect
   - [ ] No orphaned products (in RC but not in ASC)
   - [ ] No missing products (in ASC but not in RC)

3. **Offering Configuration**
   - [ ] Current offering is set (`is_current: true`)
   - [ ] Packages have products attached
   - [ ] Products in packages exist in App Store Connect

4. **Entitlement Mapping**
   - [ ] Entitlements have products attached
   - [ ] All subscription products grant an entitlement

### RevenueCat Report Section

Add to output:

```
🔗 REVENUECAT CROSS-CHECK
------------------------------------------
RC Project: {project_name}
iOS App: {app_name} ({bundle_id})

Products:
  ASC  ↔  RC
  ✓ com.myapp.premium_monthly - Matched
  ✓ com.myapp.premium_yearly - Matched
  ✗ com.myapp.lifetime - In RC but NOT in ASC (BLOCKING)

Offerings:
  ✓ Current offering set: "default"
  ✓ Packages configured with products

Entitlements:
  ✓ "Premium" linked to 2 products
```

### 5. Paywall Implementation Check

Determine which paywall approach is used and validate accordingly:

#### Option A: RevenueCat Paywalls (Built-in)

Check if using RC's native paywall feature:

- Navigate to Offerings → select current offering → "Paywall" tab
- If paywall is configured:
  - [ ] Paywall template selected
  - [ ] Localised strings configured
  - [ ] Products displayed match offering packages
  - [ ] Call-to-action text is appropriate

#### Option B: Custom Paywall Implementation (Code)

If NOT using RC Paywalls, search codebase for paywall implementation:

```typescript
// Search for RevenueCat SDK usage patterns
Grep({
  pattern: "getOfferings|useOfferings|Purchases.getOfferings",
  type: "ts",
});
Grep({ pattern: "purchasePackage|purchaseProduct", type: "ts" });
Grep({ pattern: "RevenueCatUI|PaywallView|Paywall", type: "ts" }); // RC Paywalls SDK
```

Validate custom paywall code:

1. **Offering Fetch**
   - [ ] Fetches current offering (`offerings.current`)
   - [ ] Handles null/undefined offerings gracefully
   - [ ] Shows loading state while fetching

2. **Package Display**
   - [ ] Displays packages from offering (not hardcoded products)
   - [ ] Shows correct price from `package.product.priceString`
   - [ ] Shows correct duration/period
   - [ ] Handles free trials correctly (`introPrice`)

3. **Purchase Flow**
   - [ ] Calls `purchasePackage()` not `purchaseProduct()`
   - [ ] Handles purchase errors (cancelled, failed, etc.)
   - [ ] Restores purchases available
   - [ ] Updates UI after successful purchase

4. **Product ID Consistency**
   - [ ] Product IDs in code match RC configuration
   - [ ] No hardcoded product IDs that bypass offerings
   - [ ] Platform-specific products handled (iOS vs Android)

Common code issues:

- Hardcoded product IDs instead of using offerings
- Missing error handling for failed purchases
- Not refreshing customer info after purchase
- Using wrong package identifier (e.g., `$rc_monthly` vs `$rc_annual`)
- Trial text shown but no trial configured in ASC

Report format:

```
💳 PAYWALL IMPLEMENTATION CHECK
------------------------------------------
Type: Custom Implementation (not RC Paywalls)

Code Analysis:
  Offering fetch: src/screens/PaywallScreen.tsx:45
    ✓ Uses offerings.current
    ✓ Handles loading state

  Package display: src/components/SubscriptionOption.tsx:23
    ✓ Uses package.product.priceString
    ⚠️  Trial text hardcoded - verify matches ASC config

  Purchase flow: src/hooks/usePurchase.ts:67
    ✓ Uses purchasePackage()
    ✓ Error handling present
    ✓ Restore purchases implemented

Product ID Cross-Check:
  Code Reference             RC Product              ASC Product
  "com.myapp.monthly"        ✓ Found                 ✓ Found
  "com.myapp.yearly"         ✓ Found                 ✓ Found
```

### Common RC/ASC Mismatches

- Product ID typo (e.g., `myapp_monthly` vs `myapp_premium_monthly`)
- Product exists in RC but not created/approved in ASC
- Offering references products from wrong app (Android vs iOS)
- Test store products mixed with production products
- Paywall shows trial but no trial configured in App Store Connect
- Price displayed doesn't match actual ASC price (stale cache)

### RevenueCat Browser Fallback

If RevenueCat MCP is NOT available, use Chrome browser automation instead:

1. **Navigate to RevenueCat Dashboard**

   ```
   https://app.revenuecat.com/projects/{project_id}/apps
   ```

   Ask the user for their RevenueCat project ID if not known.

2. **Check Authentication**
   If login screen shown, display:

   ```
   🔐 REVENUECAT LOGIN REQUIRED

   Please log in to RevenueCat in your browser, then run this command again.

   URL: https://app.revenuecat.com
   ```

3. **Browser Validation Steps**

   a. Navigate to Apps page - verify iOS app exists
   b. Navigate to Products page - screenshot and extract product IDs
   c. Navigate to Entitlements page - verify products attached
   d. Navigate to Offerings page - verify current offering set

4. **Cross-Reference with ASC**
   Compare product IDs found in RC browser with those in App Store Connect:
   - Extract `store_identifier` values from RC products page
   - Match against subscription product IDs in ASC

Report format (browser fallback):

```
🔗 REVENUECAT CROSS-CHECK (via browser)
------------------------------------------
RC Project: MyApp (proj1a2b3c4d)
iOS App: MyApp (App Store) - com.example.myapp

Products Found:
  RC Product ID              ASC Match
  com.myapp.premium_monthly  ✓ Found in ASC
  com.myapp.premium_yearly   ✓ Found in ASC

Offerings:
  ✓ "default" is current offering

Entitlements:
  ✓ "Premium" configured
```

## Checklist Workflow

Navigate through App Store Connect sections and validate each area.

### 1. Version Page Check (Guideline 2.3 - Accurate Metadata)

Navigate to: `/distribution/ios/version/inflight`

**Build:**

- [ ] Build uploaded and selected
- [ ] Build processing complete (no "Processing" status)

**Screenshots (must show app in use, not just splash/login):**

- [ ] iPhone 6.9" (Pro Max) screenshots present
- [ ] iPhone 6.7" screenshots present
- [ ] iPhone 6.5" screenshots present
- [ ] iPhone 5.5" screenshots present
- [ ] iPad Pro 12.9" screenshots present (if universal app)
- [ ] Screenshots show actual app functionality (not title art, login page, or splash screens)
- [ ] Screenshots content appropriate for 4+ age rating (even if app rated higher)
- [ ] If IAP exists: Screenshots/description clearly indicate which items require additional purchase

**App Previews (if present):**

- [ ] Uses only video screen captures of the app itself
- [ ] No non-app footage or stock video

**Metadata:**

- [ ] App name ≤30 characters
- [ ] Description filled (not placeholder text)
- [ ] Description does not contain trademarked terms, competitor names, or pricing info
- [ ] Keywords configured and relevant to app
- [ ] Subtitle provides context (no pricing, no unverifiable claims)
- [ ] Support URL present
- [ ] Version number set
- [ ] Copyright information present

**What's New:**

- [ ] "What's New" text describes changes for this version
- [ ] Significant new features listed specifically (not just "bug fixes")

### 2. In-App Purchases Check (Guideline 3.1.1)

Navigate to: `/distribution/subscriptions`

**Subscription Group Status:**

For each subscription group:

- [ ] Subscriptions exist in "Ready to Submit" status
- [ ] NOT "Missing Metadata" status
- [ ] NOT "Developer Action Needed" status

**Version Attachment (CRITICAL - common rejection cause):**

Return to version page and scroll to "In-App Purchases and Subscriptions":

- [ ] Subscriptions are SELECTED (not empty section)
- [ ] Count matches expected products
- [ ] All intended products are checked

**Individual Product Validation:**

Click into each subscription/IAP to verify:

- [ ] Review screenshot uploaded (required for new products)
- [ ] Screenshot shows the product in context within the app
- [ ] Display name appropriate for public audience
- [ ] Description clear and accurate
- [ ] Pricing configured for all intended territories

**Subscription-Specific Requirements (Guideline 3.1.2):**

- [ ] Subscription period ≥7 days minimum
- [ ] Clear description of what subscriber receives (how many issues, how much storage, etc.)
- [ ] Free trial properly configured in App Store Connect (if offered)
- [ ] Trial naming follows convention if applicable ("XX-day Trial")
- [ ] Upgrade/downgrade paths configured to prevent duplicate subscriptions

**Consumables & Non-Consumables:**

- [ ] Consumable IAPs do not expire (credits, gems, currencies purchased cannot expire)
- [ ] Restore purchases mechanism implemented for restorable items

**Loot Boxes / Randomized Items (if applicable):**

- [ ] Odds of receiving each item type disclosed before purchase

### 3. App Review Information Check (Guideline 2.3)

On version page, scroll to "App Review Information":

**Contact Information:**

- [ ] Contact name provided
- [ ] Contact phone provided (valid, reachable)
- [ ] Contact email provided (valid, monitored)

**Demo Account (CRITICAL if app requires login):**

- [ ] Sign-in credentials provided if app requires login
- [ ] Demo credentials actually work (test before submission)
- [ ] Demo account has access to premium features being reviewed

**Notes for Review:**

- [ ] Notes provided describing any non-obvious features
- [ ] All NEW features described specifically (not generic descriptions)
- [ ] Special requirements explained (location-based, hardware-specific, etc.)
- [ ] If app has hidden/advanced features: Explain how to access them
- [ ] If app requires specific test data: Provide it

### 4. App Information Check (Guideline 2.3)

Navigate to: General > App Information

**Privacy (CRITICAL):**

- [ ] Privacy Policy URL configured in App Store Connect
- [ ] Privacy Policy URL also accessible within the app itself (both required)

**Categorization:**

- [ ] Primary category selected and appropriate (see Apple's Category Definitions)
- [ ] Secondary category (optional - note if missing)
- [ ] Category matches actual app functionality

**Age Rating:**

- [ ] Age rating questionnaire completed honestly
- [ ] App icon/screenshots appropriate for 4+ rating (regardless of app's actual rating)
- [ ] If app includes media (films, music, games): Local content rating requirements met

**Content Rights:**

- [ ] Rights secured for all materials in icons, screenshots, and previews
- [ ] No real people's data displayed - use fictional account information

### 5. App Privacy Check (Guideline 5.1.1)

Navigate to: App Store > App Privacy

**Privacy Labels:**

- [ ] Privacy responses submitted (not "Get Started" button visible)
- [ ] Data collection types declared accurately
- [ ] Data usage purposes specified for each data type
- [ ] Third-party data sharing disclosed if applicable
- [ ] Privacy labels match actual app behaviour (will be verified by Apple)

### 6. Pricing Check

Navigate to: `/distribution/pricing`

Validate:

- [ ] Base price set (or Free)
- [ ] Availability configured

### 7. URL Validation

Test that all configured URLs are reachable using WebFetch:

```typescript
// Test each URL returns 200 (not 404, 500, or redirect to error page)
WebFetch({
  url: privacyPolicyUrl,
  prompt: "Does this page load successfully? Return YES or NO.",
});
WebFetch({
  url: supportUrl,
  prompt: "Does this page load successfully? Return YES or NO.",
});
WebFetch({
  url: marketingUrl,
  prompt: "Does this page load successfully? Return YES or NO.",
});
```

URLs to validate:

- [ ] Privacy Policy URL - Must be HTTPS, must load
- [ ] Support URL - Must be HTTPS, must load
- [ ] Marketing URL (if set) - Must be HTTPS, must load

Common URL issues:

- HTTP instead of HTTPS (Apple requires HTTPS)
- 404 page (URL doesn't exist)
- Redirect to homepage (path incorrect)
- Domain expired or DNS failure
- Page requires login (not publicly accessible)

Report format:

```
🌐 URL VALIDATION
------------------------------------------
Privacy Policy: https://example.com/privacy
  ✓ Reachable (200 OK)

Support URL: https://example.com/support
  ✗ FAILED - 404 Not Found (BLOCKING)

Marketing URL: https://example.com
  ✓ Reachable (200 OK)
```

### 8. Code Verification (Guideline 3.1.1 & 3.1.2)

Search codebase for compliance with IAP requirements:

```typescript
// Check for restore purchases implementation
Grep({ pattern: "restorePurchases|restoreTransactions", type: "ts" });

// Check for privacy policy link in-app
Grep({ pattern: "privacy|privacyPolicy|privacy-policy", type: "ts" });

// Check for license key / QR code payment bypass (FORBIDDEN)
Grep({
  pattern: "licenseKey|license_key|activationCode|qrCode.*payment",
  type: "ts",
});
```

**Required in Code:**

- [ ] Restore purchases button/function implemented
- [ ] Privacy policy accessible from within app (not just App Store)
- [ ] No alternative payment mechanisms bypassing IAP

**If Free Trial in Paywall:**

- [ ] Trial duration matches App Store Connect configuration
- [ ] Trial terms clearly communicated before starting
- [ ] Content/services that become unavailable after trial listed

Report format:

```
💻 CODE VERIFICATION
------------------------------------------
Restore Purchases:
  ✓ Found: src/screens/SettingsScreen.tsx:145
  ✓ Found: src/hooks/usePurchases.ts:78

Privacy Policy (In-App):
  ✓ Found: src/screens/SettingsScreen.tsx:234
    Links to: https://example.com/privacy

Alternative Payment Bypass:
  ✓ No forbidden patterns found

Trial Configuration:
  ⚠️  Trial text in code says "7 days" - verify matches ASC config
```

## Output Format

Generate a structured report:

```
============================================
APP STORE CONNECT PRE-FLIGHT CHECK
============================================
App: {app_name}
Version: {version}
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
- Version metadata complete
- Build uploaded and selected
- [etc.]

📋 MANUAL VERIFICATION NEEDED
------------------------------------------
- Demo account credentials work
- Screenshots accurately represent app
- [etc.]

============================================
RECOMMENDATION: [Ready to submit / Fix X issues first]
============================================
```

## Common Rejection Reasons to Check

Based on Apple's App Review Guidelines:

### Guideline 2.1 - App Completeness

- [ ] IAP products submitted with version (not just configured)
- [ ] Build does not crash or have obvious bugs
- [ ] No placeholder content (Lorem ipsum, "Coming Soon", etc.)
- [ ] All advertised features are functional

### Guideline 2.3 - Accurate Metadata

- [ ] Screenshots show actual app functionality
- [ ] App name ≤30 characters
- [ ] No competitor names or trademarked terms in metadata
- [ ] No pricing information in description/keywords
- [ ] App icon appropriate for 4+ age rating
- [ ] All new features described specifically in Review Notes
- [ ] No hidden or undocumented features
- [ ] If IAP exists: Clearly indicated in description/screenshots

### Guideline 3.1.1 - In-App Purchase

- [ ] All feature unlocks use IAP (no license keys, QR codes, cryptocurrencies)
- [ ] Restore purchases mechanism implemented
- [ ] Loot box odds disclosed before purchase (if applicable)
- [ ] Consumable currencies do not expire

### Guideline 3.1.2 - Subscriptions

- [ ] Subscription period ≥7 days
- [ ] Clear description of what subscriber receives
- [ ] Upgrade/downgrade prevents duplicate subscriptions
- [ ] Free trial configured via App Store Connect (if offered)
- [ ] No misleading trial/pricing language

### Guideline 5.1.1 - Data Collection

- [ ] Privacy labels match actual data collection
- [ ] Privacy policy accessible in App Store Connect AND in-app

### Sign-in & Demo Access

- [ ] Demo credentials provided if app requires login
- [ ] Demo credentials actually work
- [ ] Demo account has access to premium features

## Speed Optimisations

- Use `haiku` model for fast validation
- Take screenshots at key checkpoints only
- Don't read full page content - just validate presence/absence
- Parallel checks where possible (read multiple sections)

## Error Handling

If a section fails to load or is inaccessible:

- Note as "Unable to verify: [reason]"
- Continue with other checks
- Include in final report

If logged out:

- Stop immediately
- Report: "Authentication required - please log in to App Store Connect"
