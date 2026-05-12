# MDC Deployment Skill

## Description
Microsoft Defender for Cloud (MDC) deployment workshop skill. Guides IT administrators through a 5-phase structured workshop to assess licensing, enable Defender plans, discover and onboard servers, review security recommendations, and summarize posture.

## Trigger
Use this skill when the user mentions any of the following:
- "Defender for Cloud" or "MDC"
- "protect my cloud" or "cloud security"
- "CSPM" or "cloud security posture"
- "server protection" or "protect my servers"
- "Defender plans" or "enable Defender"
- "Secure Score"
- "security recommendations"

## Prerequisites
- Azure CLI (`az`) installed and authenticated via `az login`
- `az graph` extension installed (`az extension add --name resource-graph`)
- Owner or Security Admin role on target Azure subscriptions
- For the demo: MDC lab resources deployed via ARM template

## Capabilities

### Phase 1: License Assessment
Checks tenant licenses via Microsoft Graph API and maps SKUs to Defender for Cloud entitlements. Explains what's included (P1 with E5) vs. what costs extra (P2 upgrade).

### Phase 2: Plan Enablement
Queries Defender plan status across all subscriptions using Azure Resource Graph. Identifies gaps where licenses are purchased but plans are not enabled. Enables plans with customer approval.

### Phase 3: Server Discovery & Onboarding
Discovers all servers (Azure VMs + Arc-connected on-prem) and checks MDE agent status. Identifies unprotected servers and guides onboarding via Arc or direct MDE.

### Phase 4: Recommendations Review
Retrieves Secure Score, unhealthy recommendations, and score controls. Prioritizes fixes by impact (weight × affected resources).

### Phase 5: Posture Summary
Generates a complete security posture dashboard: Defender plan status, server coverage, MDE agent health, and Secure Score trend.

## Reference Files
- `agents/mdc-deployment.md` — Agent persona and behavior
- `.github/instructions/mdc/mdc-workshop-phases.instructions.md` — Phase-by-phase guide with CLI commands
- `.github/instructions/mdc/mdc-license-mapping.instructions.md` — SKU to Defender plan mapping
- `.github/instructions/mdc/mdc-pricing.instructions.md` — Pricing reference and calculator
- `.github/instructions/mdc/mdc-arg-queries.instructions.md` — ARG query reference

## Example Interaction

**User**: "I want to use MDC to protect my cloud environments, help me plan and deploy"

**Agent**:
1. Starts with Phase 1 — checks licenses: `az rest --method GET --url "https://graph.microsoft.com/v1.0/subscribedSkus"`
2. Maps SKUs to Defender entitlements, presents what's included
3. Moves to Phase 2 — checks plan status: `az graph query -q "securityresources | where type == 'microsoft.security/pricings' ..."`
4. Shows gap: "You have licenses but plans are OFF — you're paying for zero protection"
5. Asks for approval to enable plans, explains pricing
6. Continues through Phases 3-5
