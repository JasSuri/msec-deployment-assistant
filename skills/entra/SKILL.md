---
name: entra-deployment
description: Microsoft Entra identity deployment workshop for licensing, Conditional Access, MFA, and identity posture. Use when the user asks about Entra, Azure AD, MFA, SSO, or identity security rollout.
---

# Entra Deployment Skill

## Description
Microsoft Entra (Azure AD) identity and access management deployment workshop. Guides IT admins through tenant assessment, Conditional Access configuration, MFA rollout, Identity Protection, and identity security posture.

## Trigger
Use this skill when the user mentions:
- "Entra", "Azure AD", "Azure Active Directory"
- "identity", "identity security", "zero trust identity"
- "Conditional Access", "CA policies"
- "MFA", "multi-factor authentication", "passwordless"
- "SSO", "single sign-on"
- "Identity Protection", "risky sign-ins"

## Prerequisites
- Azure CLI (`az`) installed and authenticated via `az login`
- Global Reader, Security Reader, or Security Admin role in Entra ID
- For changes: Conditional Access Administrator, Authentication Policy Administrator, or Global Administrator

## Capabilities

### Phase 1: Tenant & License Assessment
Checks Entra tenant details, license SKUs (P1 vs P2), and hybrid identity status (Azure AD Connect / Cloud Sync).

### Phase 2: Security Defaults & Conditional Access
Audits Security Defaults status and existing Conditional Access policies. Identifies gaps like legacy auth not blocked, MFA not enforced, no device compliance policies.

### Phase 3: MFA & Authentication Methods
Checks MFA registration status, reviews authentication methods policy, identifies users with no MFA — the highest-risk group.

### Phase 4: Identity Protection & Risk Policies
Checks Identity Protection (P2), reviews user risk and sign-in risk policies, shows recent risky sign-ins.

### Phase 5: Posture Summary
Pulls identity-related Secure Score controls, summarizes protection status, and provides improvement roadmap.

## Reference Files
- `agents/entra-deployment.md` — Agent persona and behavior
- `.github/instructions/entra/entra-workshop-phases.instructions.md` — Phase-by-phase guide
- `.github/instructions/entra/entra-license-mapping.instructions.md` — SKU to Entra feature mapping

## Example Interaction

**User**: "Help me set up Entra and enforce MFA"

**Agent**:
1. Phase 1 — checks licenses, confirms P1/P2 entitlements
2. Phase 2 — audits Conditional Access, checks if Security Defaults is on
3. Phase 3 — checks MFA registration, identifies unregistered users
4. Creates CA policy requiring MFA for all users (with customer approval)
