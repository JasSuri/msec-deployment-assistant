---
name: intune-deployment
description: Microsoft Intune deployment workshop for endpoint management, enrollment, compliance, and app protection. Use when the user asks about Intune, MDM, MAM, Autopilot, or device management.
---

# Intune Deployment Skill

## Description
Microsoft Intune endpoint management deployment workshop. Guides IT admins through license assessment, device enrollment, compliance policies, app management, and endpoint security posture.

## Trigger
Use this skill when the user mentions:
- "Intune", "endpoint management", "device management"
- "MDM", "mobile device management"
- "MAM", "mobile app management", "app protection"
- "Autopilot", "device enrollment"
- "compliance policy", "device compliance"
- "manage my devices", "BYOD"

## Prerequisites
- Azure CLI (`az`) installed and authenticated via `az login`
- Intune Administrator or Global Administrator role
- For read-only assessment: Global Reader + Intune Reader

## Capabilities

### Phase 1: License & Tenant Assessment
Checks licenses for Intune entitlements (M365 E3/E5, EMS E3/E5, Intune standalone), current MDM authority, and enrolled device counts.

### Phase 2: Enrollment & Device Inventory
Reviews device enrollment status across platforms (Windows, macOS, iOS, Android), enrollment profiles, restrictions, and Autopilot/ADE configuration.

### Phase 3: Compliance & Configuration Policies
Audits compliance policies (BitLocker, OS version, antivirus), configuration profiles, and Conditional Access integration for blocking non-compliant devices.

### Phase 4: App Management & Protection
Checks app deployment status, App Protection Policies (MAM) for BYOD scenarios, and data loss prevention controls (cut/copy/paste, save-to restrictions).

### Phase 5: Posture Summary & Reporting
Pulls compliance dashboard: compliant vs non-compliant, platform breakdown, top compliance failures, and next steps.

## Reference Files
- `agents/intune-deployment.md` — Agent persona and behavior
- `.github/instructions/intune/intune-workshop-phases.instructions.md` — Phase-by-phase guide
- `.github/instructions/intune/intune-license-mapping.instructions.md` — SKU to Intune feature mapping

## Example Interaction

**User**: "I want to manage my company devices with Intune"

**Agent**:
1. Phase 1 — checks licenses, confirms Intune entitlement
2. Phase 2 — discovers enrolled and unenrolled devices
3. Phase 3 — reviews compliance policies, identifies non-compliant devices
4. Recommends Autopilot for new device provisioning
