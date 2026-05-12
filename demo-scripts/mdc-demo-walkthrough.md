# MDC Demo — Recording Walkthrough

## Overview
This is the talk track and step-by-step guide for recording the MDC deployment demo. The demo shows an IT admin asking GitHub Copilot to help them deploy Microsoft Defender for Cloud.

## Pre-Recording Checklist

- [ ] Azure subscription ready with lab resources deployed ([ARM template](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Security-Center%2Fmaster%2FLabs%2FFiles%2Flabdeploy.json))
- [ ] `az login` completed and authenticated to the demo subscription
- [ ] `az extension add --name resource-graph` installed
- [ ] VS Code open with this folder (`security-deployment-demos/`) as workspace
- [ ] GitHub Copilot Chat extension installed and active
- [ ] Screen recording tool ready (OBS / PowerPoint / Teams)
- [ ] Terminal font size increased for readability (14pt+)
- [ ] No sensitive data visible (other tabs, notifications, etc.)

## Demo Script

### Opening (30 seconds)
> "Today I'm going to show you how an IT admin can use AI — specifically GitHub Copilot — to plan and deploy Microsoft Defender for Cloud. No prior MDC expertise needed."

### Step 1: The Ask (15 seconds)
Open Copilot Chat and type:

```
I want to use MDC to protect my cloud environments, help me plan and deploy
```

> "I'm asking Copilot the same question a customer would ask. Let's see what happens."

### Step 2: License Check — Phase 1 (60 seconds)
Copilot should start with license assessment. Let it run the commands.

**Key points to narrate:**
- "Copilot first checks what licenses I already have — it knows that M365 E5 includes Defender for Servers P1 at no cost"
- "It shows me exactly what I'm entitled to and what I'd need to pay extra for"
- "This is the #1 confusion point for customers — server vs. device licensing"

### Step 3: Enablement Check — Phase 2 (60 seconds)
Copilot checks if Defender plans are actually enabled.

**Key points to narrate:**
- "Now it's checking across all my subscriptions whether Defender plans are actually turned on"
- "This is the most common finding — licenses purchased but plans NOT enabled, meaning zero protection"
- "It asks for my approval before enabling anything, and shows me the cost impact"

**When Copilot asks to enable plans:** Say "Yes, enable Defender for Servers and CSPM"

### Step 4: Server Discovery — Phase 3 (60 seconds)
Copilot discovers servers and checks agent status.

**Key points to narrate:**
- "It finds all my servers — both Azure VMs and any on-prem servers connected via Arc"
- "Shows me which have the MDE agent installed and which are unprotected"
- "For unprotected servers, it offers to deploy the agent automatically"

### Step 5: Recommendations — Phase 4 (45 seconds)
Copilot shows Secure Score and top recommendations.

**Key points to narrate:**
- "Now that Defender is enabled, it shows my Secure Score and top recommendations"
- "It prioritizes by impact — the items that will improve my score the most"
- "A score of 30-40% is normal on day one — the important thing is the trend"

### Step 6: Summary — Phase 5 (30 seconds)
Copilot gives the final posture summary.

**Key points to narrate:**
- "Finally, I get a clear dashboard I could share with my CISO"
- "Protected vs. exposed, score trajectory, and remaining action items"

### Closing (15 seconds)
> "In about 5 minutes, Copilot assessed my licenses, enabled Defender plans, discovered my servers, and gave me a prioritized security roadmap. No MDC expertise required."

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Copilot doesn't pick up MDC context | Ensure this folder is the VS Code workspace root |
| `az graph query` fails | Run `az extension add --name resource-graph` |
| No resources found | Ensure ARM template deployment completed (~10 min) |
| Defender plans show as "Free" | Expected pre-enablement — this is the demo flow |
| Copilot asks about wrong product | Re-prompt with "I specifically want Microsoft Defender for Cloud" |

## Reset for Re-Recording

To reset the demo environment:
```bash
# Disable all Defender plans (back to Free tier)
SUB_ID=$(az account show --query id -o tsv)
for plan in VirtualMachines CloudPosture SqlServers StorageAccounts AppServices KubernetesService KeyVaults Arm Dns; do
  az security pricing create --name $plan --tier Free --subscription $SUB_ID
done

# Verify reset
az graph query -q "securityresources | where type == 'microsoft.security/pricings' | project name, tier=tostring(properties.pricingTier)" --output table
```
