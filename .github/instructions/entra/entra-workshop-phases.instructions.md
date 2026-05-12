# Entra Deployment Workshop — Phase-by-Phase Guide

This file provides the structured workshop flow for deploying Microsoft Entra identity security. Follow the phases in order.

---

## Phase 1: Tenant & License Assessment — "What's my identity posture today?"

**Goal:** Understand the customer's Entra tenant, licensing, and hybrid identity setup.

### CLI Commands

**Check tenant details:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --query "value[0].{TenantId:id, Name:displayName, Verified:verifiedDomains[0].name}" \
  --output table
```

**Check licenses for Entra entitlements:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --query "value[?consumedUnits > \`0\`].{SKU:skuPartNumber, Total:prepaidUnits.enabled, Used:consumedUnits}" \
  --output table
```

**Check Azure AD Connect / Cloud Sync status:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --query "value[0].{DirSync:onPremisesSyncEnabled, LastSync:onPremisesLastSyncDateTime}"
```

**Count users:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/users/\$count" \
  --headers "ConsistencyLevel=eventual"
```

### How to Present Results

```
🏢 YOUR ENTRA TENANT
 Tenant: Contoso (contoso.com)
 Users: 2,450
 Directory Sync: ✅ Enabled (last sync: 2 hours ago)

📦 YOUR LICENSES
 ✅ M365 E5 (200 seats) → Entra ID P2 INCLUDED
 ✅ Entra ID P2 gives you: Conditional Access, Identity Protection, PIM, access reviews

💡 You have full Entra ID P2 — let's make sure it's configured properly.
```

---

## Phase 2: Security Defaults & Conditional Access — "Am I protected?"

**Goal:** Check baseline identity protection. Security Defaults is the minimum; Conditional Access is the proper approach.

### CLI Commands

**Check if Security Defaults is enabled:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/policies/identitySecurityDefaultsEnforcementPolicy" \
  --query "{Enabled:isEnabled}"
```

**List Conditional Access policies:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" \
  --query "value[].{Name:displayName, State:state, GrantControls:grantControls.builtInControls}"
```

**Check named locations (trusted IPs):**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/identity/conditionalAccess/namedLocations" \
  --query "value[].{Name:displayName, Type:['@odata.type']}"
```

### How to Present Results

```
🔐 IDENTITY PROTECTION STATUS

Security Defaults: ❌ Disabled
Conditional Access Policies: 3 found

 Policy                        | State    | Controls
 Require MFA for admins        | Enabled  | MFA
 Block legacy auth              | Enabled  | Block
 Require compliant device       | Report   | CompliantDevice

⚠️ "Require compliant device" is in report-only mode — not enforcing yet.
❌ No policy requiring MFA for ALL users — only admins are covered.

💡 RECOMMENDATION
 1. Create a CA policy requiring MFA for all users (exclude break-glass accounts)
 2. Move "Require compliant device" from Report to Enabled
 3. Verify legacy auth is fully blocked
```

### Enable Conditional Access (with customer approval)

**Create a CA policy requiring MFA for all users:**
```bash
az rest --method POST \
  --url "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" \
  --body '{
    "displayName": "Require MFA for all users",
    "state": "enabledForReportingButNotEnforced",
    "conditions": {
      "users": { "includeUsers": ["All"], "excludeUsers": ["<break-glass-id>"] },
      "applications": { "includeApplications": ["All"] }
    },
    "grantControls": {
      "operator": "OR",
      "builtInControls": ["mfa"]
    }
  }'
```

⚠️ **Always create in Report-Only first.** Move to Enabled after validating no disruption.

---

## Phase 3: MFA & Authentication Methods — "How are my users signing in?"

**Goal:** Check MFA registration and authentication method configuration.

### CLI Commands

**Check authentication methods policy:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/policies/authenticationMethodsPolicy" \
  --query "authenticationMethodConfigurations[].{Method:id, State:state}"
```

**Get MFA registration details (per-user — paginated):**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/reports/authenticationMethods/userRegistrationDetails?\$top=50" \
  --query "value[].{User:userDisplayName, MFA:isMfaRegistered, Methods:methodsRegistered}"
```

**Count MFA registered vs not:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/reports/authenticationMethods/userRegistrationDetails?\$filter=isMfaRegistered eq false&\$count=true" \
  --headers "ConsistencyLevel=eventual" \
  --query "'@odata.count'"
```

### How to Present Results

```
🔑 AUTHENTICATION METHODS

 Method             | State
 Microsoft Auth     | ✅ Enabled
 FIDO2 Keys         | ✅ Enabled
 SMS                | ⚠️ Enabled (less secure)
 Voice              | ❌ Disabled
 Email OTP          | ✅ Enabled

📊 MFA REGISTRATION
 Registered:    2,100 / 2,450 (85.7%)
 NOT registered:  350 users ❌

⚠️ 350 users have NO MFA — they are your highest-risk group.
   If a Conditional Access policy requires MFA, these users will be
   prompted to register on next sign-in.

💡 RECOMMENDATION
 1. Enable MFA registration campaign (nudge users to register)
 2. Consider disabling SMS as an auth method (SIM swap risk)
 3. Push Microsoft Authenticator as the default method
```

---

## Phase 4: Identity Protection & Risk Policies — "What threats am I catching?"

**Goal:** Check Identity Protection (requires Entra ID P2) and risk-based policies.

### CLI Commands

**Check risk-based policies:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" \
  --query "value[?contains(displayName, 'risk')].{Name:displayName, State:state}"
```

**Get recent risky sign-ins:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/identityProtection/riskyUsers?\$top=10&\$orderby=riskLastUpdatedDateTime desc" \
  --query "value[].{User:userDisplayName, RiskLevel:riskLevel, RiskState:riskState, LastUpdated:riskLastUpdatedDateTime}"
```

**Get risky sign-in detections:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/identityProtection/riskDetections?\$top=10&\$orderby=activityDateTime desc" \
  --query "value[].{Type:riskEventType, Level:riskLevel, User:userDisplayName, IP:ipAddress, Time:activityDateTime}"
```

### How to Present Results

```
🛡️ IDENTITY PROTECTION

Risk Policies:
 ❌ No user risk policy configured
 ❌ No sign-in risk policy configured

Risky Users (last 30 days):
 User              | Risk Level | State
 john@contoso.com  | High       | atRisk
 jane@contoso.com  | Medium     | atRisk

⚠️ You have P2 licenses but Identity Protection policies are NOT configured.
   You're paying for threat detection but not acting on it.

💡 RECOMMENDATION
 1. Create CA policy: High user risk → require password change + MFA
 2. Create CA policy: Medium+ sign-in risk → require MFA
 3. Review the 2 users currently at risk and remediate
```

---

## Phase 5: Posture Summary — "How does my identity security look?"

**Goal:** Summarize identity security posture and provide ongoing recommendations.

### CLI Commands

**Get Secure Score (identity controls):**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/secureScores?\$top=1" \
  --query "value[0].controlScores[?contains(controlCategory, 'Identity')].{Control:controlName, Score:score, MaxScore:maxScore, Description:description}"
```

### How to Present Results

```
🛡️ IDENTITY SECURITY SUMMARY

 Area                    | Status
 MFA for admins          | ✅ Enforced
 MFA for all users       | ⚠️ Report-only (not enforcing)
 Legacy auth blocked     | ✅ Blocked
 Identity Protection     | ❌ Not configured
 MFA registration        | ⚠️ 85.7% (350 users exposed)
 Conditional Access       | ⚠️ 3 policies (gaps exist)
 PIM                     | ❌ Not configured

🎯 TOP 3 ACTIONS
 1. Enable MFA CA policy for all users (move from Report to Enabled)
 2. Configure Identity Protection risk policies
 3. Set up PIM for privileged role management
```
