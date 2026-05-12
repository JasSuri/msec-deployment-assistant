# Purview Deployment Workshop — Phase-by-Phase Guide

This file provides the structured workshop flow for deploying Microsoft Purview data security and compliance. Follow the phases in order.

---

## Phase 1: License & Compliance Assessment — "What data protection do I have?"

**Goal:** Understand the customer's Purview licensing and current compliance baseline.

### CLI Commands

**Check licenses:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --query "value[?consumedUnits > \`0\`].{SKU:skuPartNumber, Total:prepaidUnits.enabled, Used:consumedUnits}" \
  --output table
```

**Check Compliance Manager score (PowerShell):**
```powershell
# Connect to Security & Compliance
Connect-IPPSSession

# Get compliance score overview
Get-ComplianceSecurityFilter | Select-Object Name, Action
```

### How to Present Results

```
📦 YOUR PURVIEW LICENSES
 ✅ M365 E5 (200 seats) → Full Purview features included
   • Sensitivity labels + auto-labeling
   • DLP (Exchange, SPO, OneDrive, Teams, Endpoint)
   • Insider Risk Management
   • eDiscovery Premium
   • Compliance Manager

💡 You have full E5 — let's see what's configured.
```

---

## Phase 2: Data Classification & Sensitivity Labels — "Do I know where my sensitive data is?"

**Goal:** Check if data is being classified and protected with sensitivity labels.

### CLI Commands

**List sensitivity labels:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/security/informationProtection/sensitiveTypes" \
  --query "value[].{Name:name, ID:id}" \
  --output table
```

**Check label policies (PowerShell):**
```powershell
Connect-IPPSSession

# List sensitivity labels
Get-Label | Select-Object Name, DisplayName, Priority, Disabled

# List label policies
Get-LabelPolicy | Select-Object Name, Labels, ExchangeLocation

# List auto-labeling policies
Get-AutoSensitivityLabelPolicy | Select-Object Name, Mode, Priority
```

### How to Present Results

```
🏷️ DATA CLASSIFICATION STATUS

 Sensitivity Labels: 5 configured
  Label              | Encryption | Auto-label
  Public             | ❌          | ❌
  Internal           | ❌          | ❌
  Confidential       | ✅          | ⚠️ Simulation only
  Highly Confidential| ✅          | ❌
  Restricted         | ✅          | ❌

 Published to: All users ✅
 Auto-labeling: ⚠️ 1 policy in simulation mode, none enforcing

💡 RECOMMENDATION
 1. Review auto-labeling simulation results
 2. Enable auto-labeling for "Confidential" if simulation shows accuracy
 3. Consider auto-labeling for credit card numbers, SSNs in documents
```

---

## Phase 3: Data Loss Prevention (DLP) — "Am I preventing data leaks?"

**Goal:** Review DLP policies and identify unprotected channels.

### CLI Commands (PowerShell)

```powershell
Connect-IPPSSession

# List DLP policies
Get-DlpCompliancePolicy | Select-Object Name, Mode, ExchangeLocation, SharePointLocation, OneDriveLocation, TeamsLocation

# List DLP rules
Get-DlpComplianceRule | Select-Object Name, ParentPolicyName, BlockAccess, NotifyUser

# Check DLP activity
Get-DlpDetailReport -StartDate (Get-Date).AddDays(-30) -EndDate (Get-Date) | Group-Object PolicyName | Select-Object Name, Count
```

### How to Present Results

```
🛡️ DLP STATUS

 Policy               | Exchange | SPO | OneDrive | Teams | Endpoint
 PII Protection       | ✅        | ✅   | ✅        | ❌     | ❌
 Financial Data       | ✅        | ✅   | ✅        | ❌     | ❌
 Health Records       | ✅        | ❌   | ❌        | ❌     | ❌

 DLP matches (last 30 days): 247
 DLP blocks (last 30 days): 18

❌ GAPS:
 • Teams NOT covered by any DLP policy — users can share PII in chat
 • Endpoint DLP not enabled — data can be copied to USB/printed
 • Health Records policy only covers Exchange

💡 RECOMMENDATION
 1. Extend PII and Financial Data policies to Teams
 2. Enable Endpoint DLP for USB, print, and clipboard controls
 3. Extend Health Records policy to SharePoint and OneDrive
```

---

## Phase 4: Data Lifecycle & Insider Risk — "Am I managing data retention and insider threats?"

**Goal:** Review retention policies and insider risk configuration.

### CLI Commands (PowerShell)

```powershell
Connect-IPPSSession

# Retention policies
Get-RetentionCompliancePolicy | Select-Object Name, Enabled, ExchangeLocation, SharePointLocation, ModernGroupLocation

# Retention labels
Get-ComplianceTag | Select-Object Name, RetentionAction, RetentionDuration, IsRecordLabel

# Insider Risk policies (if configured)
# Note: Insider Risk Management is configured via the Purview portal (compliance.microsoft.com)
```

### How to Present Results

```
📂 DATA LIFECYCLE
 Retention policies: 2
  • "Keep email 7 years" → Exchange ✅
  • "Keep SPO 5 years" → SharePoint, OneDrive ✅
 Retention labels: 3 (none auto-applied)
 Records management: ❌ Not configured

🔍 INSIDER RISK
 Insider Risk Management: ❌ Not configured
 Communication Compliance: ❌ Not configured

⚠️ You have E5 licenses that include Insider Risk — it's not enabled.

💡 RECOMMENDATION
 1. Enable Insider Risk with "Data theft by departing users" template
 2. Set up Communication Compliance for regulatory requirements
 3. Consider auto-applying retention labels for known document types
```

---

## Phase 5: Compliance Score & Summary

### How to Present Results

```
📊 PURVIEW COMPLIANCE SUMMARY

 Compliance Score: 48/100

 Area                    | Status
 Sensitivity labels      | ✅ Configured (5 labels)
 Auto-labeling           | ⚠️ Simulation only
 DLP (Email)             | ✅ Active
 DLP (Teams)             | ❌ Not covered
 DLP (Endpoint)          | ❌ Not enabled
 Retention policies      | ✅ Email + SPO
 Records management      | ❌ Not configured
 Insider Risk            | ❌ Not configured
 eDiscovery              | ⚠️ Available, no active cases

🎯 TOP 3 ACTIONS
 1. Extend DLP to Teams and Endpoint
 2. Enable Insider Risk Management
 3. Enable auto-labeling for Confidential content
```
