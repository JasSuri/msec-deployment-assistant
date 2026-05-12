# Defender XDR Deployment Skill

## Description
Microsoft Defender XDR unified security deployment workshop. Guides IT admins through license assessment, Defender for Endpoint (MDE), Defender for Office 365 (MDO), Defender for Identity (MDI), Defender for Cloud Apps (MDA), and unified XDR incident management.

## Trigger
Use this skill when the user mentions:
- "Defender XDR", "Microsoft Defender", "XDR"
- "Defender for Endpoint", "MDE", "endpoint protection"
- "Defender for Office", "MDO", "email security", "anti-phishing"
- "Defender for Identity", "MDI", "identity threats"
- "Defender for Cloud Apps", "MDA", "MCAS", "shadow IT"
- "security incidents", "threat detection", "SOC"

## Prerequisites
- Azure CLI (`az`) installed and authenticated via `az login`
- Security Administrator or Global Administrator role
- For MDE: access to security.microsoft.com
- For MDI: domain controller access for sensor deployment

## Capabilities

### Phase 1: License & Product Assessment
Checks tenant licenses, maps SKUs to Defender products (MDE P1/P2, MDO P1/P2, MDI, MDA), clarifies E3 vs E5 coverage.

### Phase 2: Defender for Endpoint (MDE)
Checks device onboarding status, sensor health, antivirus configuration, attack surface reduction (ASR) rules, and automated investigation settings.

### Phase 3: Defender for Office 365 (MDO)
Reviews Safe Attachments, Safe Links, anti-phishing policies, preset security policies, and Zero-hour Auto Purge (ZAP).

### Phase 4: Defender for Identity & Cloud Apps
Checks MDI sensor deployment on DCs, reviews health alerts. Checks MDA discovered apps, connected apps, and OAuth governance.

### Phase 5: XDR Unified View & Incidents
Verifies cross-product correlation, reviews incident queue, hunting queries, custom detections, and Secure Score.

## Important Disambiguation
> **Defender for Cloud (MDC)** is a separate product for cloud workload protection (VMs, SQL, containers, CSPM). If the customer asks about MDC, route to the `mdc-deployment` agent instead.

## Reference Files
- `agents/defender-xdr-deployment.md` — Agent persona and behavior
- `.github/instructions/defender-xdr/defender-workshop-phases.instructions.md` — Phase-by-phase guide
- `.github/instructions/defender-xdr/defender-license-mapping.instructions.md` — SKU to Defender product mapping

## Example Interaction

**User**: "I want to protect my endpoints and email with Microsoft Defender"

**Agent**:
1. Phase 1 — checks licenses, confirms MDE P2 and MDO P2 entitlements
2. Phase 2 — checks MDE onboarding, identifies unprotected devices
3. Phase 3 — reviews email protection policies, identifies gaps
4. Phase 5 — shows unified incident queue and Secure Score
