# Purview Deployment Skill

## Description
Microsoft Purview data security and compliance deployment workshop. Guides IT admins and compliance officers through license assessment, sensitivity labeling, data loss prevention (DLP), data lifecycle management, insider risk, and compliance posture.

## Trigger
Use this skill when the user mentions:
- "Purview", "data security", "data protection"
- "DLP", "data loss prevention", "prevent data leaks"
- "sensitivity labels", "classification", "classify data"
- "retention", "data lifecycle", "records management"
- "insider risk", "insider threat"
- "compliance", "GDPR", "HIPAA", "regulatory"
- "eDiscovery", "legal hold"
- "Compliance Manager", "compliance score"

## Prerequisites
- Azure CLI (`az`) installed and authenticated via `az login`
- Compliance Administrator, Global Reader, or Global Administrator role
- PowerShell: `Connect-IPPSSession` for Security & Compliance Center
- For insider risk: Insider Risk Management role group

## Capabilities

### Phase 1: License & Compliance Assessment
Checks licenses for Purview feature entitlements (E3 vs E5, E5 Compliance add-on). Checks current Compliance Manager score.

### Phase 2: Data Classification & Sensitivity Labels
Reviews existing sensitivity labels, label policies, auto-labeling, sensitive information types (SITs), and trainable classifiers.

### Phase 3: Data Loss Prevention (DLP)
Audits DLP policies across Exchange, SharePoint, OneDrive, Teams, and Endpoint. Checks policy tips, alerts, and coverage gaps.

### Phase 4: Data Lifecycle & Insider Risk
Reviews retention policies and labels, records management, insider risk management policies, communication compliance, and eDiscovery.

### Phase 5: Compliance Score & Posture Summary
Pulls Compliance Manager score, shows assessment status per regulation, identifies highest-impact improvement actions.

## Reference Files
- `agents/purview-deployment.md` — Agent persona and behavior
- `.github/instructions/purview/purview-workshop-phases.instructions.md` — Phase-by-phase guide
- `.github/instructions/purview/purview-license-mapping.instructions.md` — SKU to Purview feature mapping

## Example Interaction

**User**: "We need to protect sensitive data and comply with GDPR"

**Agent**:
1. Phase 1 — checks licenses, confirms E5 Compliance entitlements
2. Phase 2 — reviews sensitivity labels, checks if data is classified
3. Phase 3 — audits DLP policies, identifies unprotected channels
4. Phase 5 — shows Compliance Manager score for GDPR assessment
