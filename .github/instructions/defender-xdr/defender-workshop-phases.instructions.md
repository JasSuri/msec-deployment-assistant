# Defender XDR Deployment Workshop — Phase-by-Phase Guide

This file provides the structured workshop flow for deploying the Microsoft Defender XDR suite. Follow the phases in order.

---

## Phase 1: License & Product Assessment — "What Defender products do I have?"

**Goal:** Understand what Defender products the customer is entitled to and what's currently enabled.

### CLI Commands

**Check licenses:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --query "value[?consumedUnits > \`0\`].{SKU:skuPartNumber, Total:prepaidUnits.enabled, Used:consumedUnits}" \
  --output table
```

### How to Present Results

```
🛡️ YOUR DEFENDER PRODUCTS

 Product              | Entitled | Status
 Defender for Endpoint | P2 ✅    | Check Phase 2
 Defender for Office   | P2 ✅    | Check Phase 3
 Defender for Identity  | ✅       | Check Phase 4
 Defender for Cloud Apps| ✅       | Check Phase 4
 Defender XDR (unified)| ✅       | Check Phase 5

💡 You have the full Defender XDR stack with E5.
   Let's verify each product is actually enabled and configured.
```

---

## Phase 2: Defender for Endpoint (MDE) — "Are my devices protected?"

**Goal:** Verify MDE onboarding, sensor health, and security configuration.

### CLI Commands

**Check MDE-onboarded devices via Graph Security API:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/microsoft.graph.security.runHuntingQuery" \
  --method POST \
  --body '{"query": "DeviceInfo | summarize arg_max(Timestamp, *) by DeviceId | summarize TotalDevices=count(), HealthyDevices=countif(SensorHealthState == \"Active\") | extend UnhealthyDevices = TotalDevices - HealthyDevices"}'
```

**Alternative — list devices with security state:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/alerts_v2?\$top=10&\$orderby=createdDateTime desc" \
  --query "value[].{Title:title, Severity:severity, Status:status, Service:serviceSource, Created:createdDateTime}" \
  --output table
```

### Key Configuration to Check

| Setting | What to Look For |
|---------|-----------------|
| Sensor health | All devices reporting "Active" |
| Antivirus | Microsoft Defender AV enabled (not third-party) |
| ASR rules | At least audit mode on critical rules |
| Automated Investigation | Enabled (full or semi-automated) |
| Tamper Protection | Enabled |
| Network Protection | Enabled |
| Web Content Filtering | Configured |

### How to Present Results

```
🖥️ DEFENDER FOR ENDPOINT STATUS

 Onboarded devices: 342 / ~400 expected
 Sensor health: 330 Active ✅, 12 Inactive ❌
 Antivirus: Microsoft Defender AV (all devices)
 ASR rules: ⚠️ Not configured

❌ 12 devices with inactive sensors — may be offline or misconfigured
⚠️ ASR rules not configured — missing critical exploit prevention

💡 RECOMMENDATION
 1. Investigate 12 inactive sensors (check connectivity)
 2. Enable ASR rules in Audit mode first:
    - Block Office apps from creating child processes
    - Block credential stealing from LSASS
    - Block executable content from email
 3. Enable Tamper Protection if not already on
```

---

## Phase 3: Defender for Office 365 (MDO) — "Is my email safe?"

**Goal:** Verify email protection configuration.

### CLI Commands

**Check via Security & Compliance PowerShell (recommended):**
```powershell
# Connect to Security & Compliance
Connect-IPPSSession

# Check Safe Attachments policies
Get-SafeAttachmentPolicy | Select-Object Name, Action, Enable

# Check Safe Links policies
Get-SafeLinksPolicy | Select-Object Name, EnableSafeLinksForEmail, EnableSafeLinksForTeams

# Check anti-phishing policies
Get-AntiPhishPolicy | Select-Object Name, EnableMailboxIntelligence, EnableSpoofIntelligence, PhishThresholdLevel
```

**Check preset security policies (Graph API):**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/attackSimulation/simulationAutomations" \
  --query "value[].{Name:displayName, Status:status}"
```

### How to Present Results

```
📧 DEFENDER FOR OFFICE 365 STATUS

 Safe Attachments:  ✅ Enabled (Dynamic Delivery)
 Safe Links:        ✅ Enabled (Email + Teams)
 Anti-Phishing:     ⚠️ Default policy only (no impersonation protection)
 Anti-Spam:         ✅ Default + custom policy
 ZAP:               ✅ Active

⚠️ No custom anti-phishing policy — executives are not protected
   against impersonation attacks.

💡 RECOMMENDATION
 1. Create anti-phishing policy with impersonation protection for executives
 2. Enable user and domain impersonation protection
 3. Consider enabling preset "Strict" security policy for high-risk users
```

---

## Phase 4: Defender for Identity & Cloud Apps — "Identity threats and shadow IT"

**Goal:** Verify MDI sensor deployment and MDA configuration.

### Key Checks

**Defender for Identity (MDI):**
- Sensor deployment on all domain controllers
- Health alerts (certificate, connectivity, performance)
- Monitored activities and detection coverage

**Defender for Cloud Apps (MDA):**
- Connected apps (M365, Azure, third-party SaaS)
- Discovered apps (shadow IT from Cloud Discovery)
- App governance policies
- OAuth app consent policies

### CLI Commands

```bash
# Check MDA connected apps
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/alerts_v2?\$filter=serviceSource eq 'microsoftCloudAppSecurity'&\$top=5" \
  --query "value[].{Title:title, Severity:severity, Status:status}"
```

### How to Present Results

```
🔍 IDENTITY & CLOUD APP PROTECTION

 Defender for Identity:
  Sensors deployed: 2 / 3 domain controllers ⚠️
  Health alerts: 1 (certificate expiring in 14 days)

 Defender for Cloud Apps:
  Connected apps: 4 (M365, Azure, Salesforce, ServiceNow)
  Discovered apps: 287 (12 flagged high risk)
  Active policies: 8

⚠️ 1 DC missing MDI sensor — blind spot for lateral movement detection
❌ 12 high-risk shadow IT apps discovered — review and sanction/block

💡 RECOMMENDATION
 1. Deploy MDI sensor to remaining DC
 2. Renew expiring certificate on sensor
 3. Review and block/sanction high-risk discovered apps
```

---

## Phase 5: XDR Unified View — "Is everything connected?"

**Goal:** Verify cross-product correlation and unified incident management.

### CLI Commands

**Get recent incidents:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/incidents?\$top=10&\$orderby=createdDateTime desc" \
  --query "value[].{Title:displayName, Severity:severity, Status:status, Created:createdDateTime, Alerts:alerts}" \
  --output table
```

**Get Secure Score:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/secureScores?\$top=1" \
  --query "value[0].{Score:currentScore, Max:maxScore, Percentage:averageComparativeScores}"
```

### How to Present Results

```
🛡️ DEFENDER XDR SUMMARY

 Product               | Status    | Coverage
 Defender for Endpoint  | ✅ Active  | 330/342 healthy
 Defender for Office    | ✅ Active  | All mailboxes
 Defender for Identity  | ⚠️ Partial | 2/3 DCs
 Defender for Cloud Apps| ✅ Active  | 4 connected apps

 Unified Incidents (last 7 days): 12
  • High severity: 2
  • Medium severity: 5
  • Low/Info: 5

 Secure Score: 62/100

🎯 TOP 3 ACTIONS
 1. Deploy MDI sensor to remaining DC
 2. Enable ASR rules (Audit → Block)
 3. Add impersonation protection for executives
```
