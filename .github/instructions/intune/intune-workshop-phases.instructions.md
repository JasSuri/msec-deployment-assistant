# Intune Deployment Workshop — Phase-by-Phase Guide

This file provides the structured workshop flow for deploying Microsoft Intune endpoint management. Follow the phases in order.

---

## Phase 1: License & Tenant Assessment — "What device management do I have?"

**Goal:** Understand the customer's Intune licensing and current MDM setup.

### CLI Commands

**Check licenses for Intune entitlements:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --query "value[?consumedUnits > \`0\`].{SKU:skuPartNumber, Total:prepaidUnits.enabled, Used:consumedUnits}" \
  --output table
```

**Check MDM authority:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --query "value[0].{MDMAuthority:mobileDeviceManagementAuthority}"
```

**Count managed devices by platform:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?\$count=true&\$top=1" \
  --headers "ConsistencyLevel=eventual" \
  --query "'@odata.count'"
```

### How to Present Results

```
📱 YOUR INTUNE SETUP
 MDM Authority: ✅ Intune
 Licensed users: 500 (M365 E5)
 Managed devices: 342

💡 You have 500 licensed users but only 342 devices enrolled.
   Let's find out what's missing.
```

---

## Phase 2: Enrollment & Device Inventory — "What devices do I have?"

**Goal:** Discover all devices, check enrollment status, and identify gaps.

### CLI Commands

**List managed devices by OS:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/managedDevices" \
  --query "value[].{Name:deviceName, OS:operatingSystem, Compliance:complianceState, Enrolled:enrolledDateTime, Owner:managedDeviceOwnerType}" \
  --output table
```

**Count devices by platform:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?\$count=true" \
  --headers "ConsistencyLevel=eventual"
```

**Check Autopilot devices:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/windowsAutopilotDeviceIdentities" \
  --query "value[].{Serial:serialNumber, Model:model, Profile:deploymentProfileAssignmentStatus}" \
  --output table
```

**Check enrollment restrictions:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/deviceEnrollmentConfigurations" \
  --query "value[].{Name:displayName, Type:deviceEnrollmentConfigurationType, Priority:priority}" \
  --output table
```

### How to Present Results

```
📊 DEVICE INVENTORY
 Platform  | Enrolled | Corporate | Personal
 Windows   | 280      | 250       | 30
 macOS     | 42       | 40        | 2
 iOS       | 15       | 5         | 10
 Android   | 5        | 0         | 5

 Autopilot devices registered: 180
 Enrollment restrictions: Default (all platforms allowed)

⚠️ 30 personal Windows devices enrolled — review BYOD policy
💡 180 Autopilot devices ready for zero-touch provisioning
```

---

## Phase 3: Compliance & Configuration — "Are my devices meeting standards?"

**Goal:** Review compliance policies and identify non-compliant devices.

### CLI Commands

**List compliance policies:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/deviceCompliancePolicies" \
  --query "value[].{Name:displayName, Platform:['@odata.type'], Created:createdDateTime}" \
  --output table
```

**Get device compliance status summary:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/deviceCompliancePolicyDeviceStateSummary" \
  --query "{Compliant:compliantDeviceCount, NonCompliant:nonCompliantDeviceCount, InGrace:inGracePeriodCount, NotEvaluated:notApplicableDeviceCount}"
```

**List configuration profiles:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceManagement/deviceConfigurations" \
  --query "value[].{Name:displayName, Platform:['@odata.type']}" \
  --output table
```

### How to Present Results

```
✅ COMPLIANCE STATUS
 Compliant:     290 devices (84.8%)
 Non-compliant:  32 devices ❌
 In grace:       10 devices ⚠️
 Not evaluated:  10 devices

❌ TOP NON-COMPLIANCE REASONS
 1. BitLocker not enabled (18 devices)
 2. OS version below minimum (8 devices)
 3. Antivirus out of date (6 devices)

💡 RECOMMENDATION
 1. Push BitLocker policy to 18 non-compliant devices
 2. Create Windows Update ring for OS patching
 3. Integrate compliance with Conditional Access to block non-compliant devices
```

---

## Phase 4: App Management — "Are my apps and data secure?"

**Goal:** Review app deployment and App Protection Policies (MAM).

### CLI Commands

**List deployed apps:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps" \
  --query "value[].{Name:displayName, Type:['@odata.type'], Publisher:publisher}" \
  --output table
```

**List App Protection Policies:**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/deviceAppManagement/managedAppPolicies" \
  --query "value[].{Name:displayName, Type:['@odata.type']}" \
  --output table
```

### How to Present Results

```
📦 APP MANAGEMENT
 Deployed apps: 12 (8 required, 4 available)
 App Protection Policies: 2 (iOS, Android)

⚠️ No App Protection Policy for Windows — data on Windows BYOD devices is unprotected
💡 Create a Windows MAM policy to protect corporate data on unmanaged Windows devices
```

---

## Phase 5: Posture Summary

```
📱 ENDPOINT MANAGEMENT SUMMARY

 Area                  | Status
 Devices enrolled      | 342 / ~500 expected (68%)
 Compliance rate       | 84.8%
 Autopilot ready       | 180 devices
 App Protection (iOS)  | ✅ Configured
 App Protection (Droid)| ✅ Configured
 App Protection (Win)  | ❌ Missing
 CA + Compliance       | ⚠️ Not integrated

🎯 TOP 3 ACTIONS
 1. Enroll remaining ~158 devices
 2. Integrate compliance with Conditional Access
 3. Create Windows App Protection Policy for BYOD
```
