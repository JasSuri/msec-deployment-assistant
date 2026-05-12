# MDC Deployment Workshop — Phase-by-Phase Guide

This file provides the structured workshop flow for deploying Microsoft Defender for Cloud with SMC customers. Follow the phases in order.

---

## Phase 1: License Clarity — "What do I actually have?"

**Goal:** Help the customer understand their Defender licensing entitlements. Most SMC customers are confused about server vs device licensing and what's included in their existing plans.

### Key Concepts to Explain

- **Server licenses ≠ device licenses.** Defender for Servers requires per-server licensing, not per-user
- **M365 E5 / E5 Security includes Defender for Servers P1** at no extra cost — most customers don't know this
- **P1 vs P2 matters** — P2 adds JIT VM access, agentless scanning, adaptive application controls, file integrity monitoring

### CLI Commands

**Check tenant licenses (Graph API via Azure CLI):**
```bash
az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --query "value[?consumedUnits > \`0\`].{SKU:skuPartNumber, Total:prepaidUnits.enabled, Used:consumedUnits}" \
  --output table
```

**List subscriptions the customer has access to:**
```bash
az account list --query "[].{Name:name, ID:id, State:state}" --output table
```

### How to Interpret Results

Map the SKUs found to Defender entitlements using the license mapping reference (`mdc-license-mapping.instructions.md`). Then present:

1. What licenses the customer HAS
2. What Defender capabilities those licenses INCLUDE
3. What capabilities they're MISSING (and what it would cost — see pricing reference)

### Example Output to Customer

```
📦 YOUR LICENSES
✅ M365 E5 Security (50 seats)
   └─ INCLUDES: Defender for Servers P1 at no extra cost
   └─ DOES NOT INCLUDE: Servers P2, paid CSPM

⚠️ SERVER vs DEVICE LICENSING
   Your E5 gives P1 per eligible server, but P2 requires a
   separate per-server add-on ($10/server/month).

💡 WITH P1 ONLY, YOU MISS:
   • Just-in-time VM access
   • Agentless vulnerability scanning
   • Adaptive application controls
   • File integrity monitoring
```

---

## Phase 2: Enablement Check — "Why isn't anything turned on?"

**Goal:** Check if Defender plans are actually enabled on the customer's Azure subscriptions. The most common finding: licenses are purchased but plans are NOT enabled — they're paying for zero protection.

### CLI Commands

**Check Defender plans across ALL subscriptions (ARG — single query):**
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/pricings'
  | extend planName = name, tier = tostring(properties.pricingTier), subPlan = tostring(properties.subPlan)
  | project subscriptionId, planName, tier, subPlan
  | order by subscriptionId asc, planName asc
" --output table
```

**Check auto-provisioning status for a specific subscription:**
```bash
az security auto-provisioning-setting list --subscription <sub-id> --output table
```

**Check specific Defender plan on a subscription:**
```bash
az security pricing show --name VirtualMachines --subscription <sub-id>
```

### Key Plans to Check

| Plan Name | What It Protects |
|-----------|-----------------|
| `VirtualMachines` | Azure VMs + Arc servers (Defender for Servers) |
| `SqlServers` | Azure SQL |
| `SqlServerVirtualMachines` | SQL on VMs |
| `StorageAccounts` | Azure Storage |
| `AppServices` | App Service |
| `KubernetesService` | AKS |
| `ContainerRegistry` | Container images |
| `KeyVaults` | Key Vault |
| `Dns` | DNS layer |
| `Arm` | Azure Resource Manager |
| `CloudPosture` | Defender CSPM |

### How to Present Results

Flag the critical gap: **"You're paying for licenses but getting ZERO protection"** if plans are not enabled. Show:
- Which subscriptions have plans ON vs OFF
- Which plans matter most for their workloads
- Auto-provisioning status (usually OFF — should be ON)

### Enable a Defender Plan (with customer approval)

```bash
# Enable Defender for Servers P2 on a subscription
az security pricing create \
  --name VirtualMachines \
  --tier Standard \
  --subscription <sub-id>

# Enable Defender CSPM
az security pricing create \
  --name CloudPosture \
  --tier Standard \
  --subscription <sub-id>

# Enable auto-provisioning for MDE agent
az security auto-provisioning-setting update \
  --name default \
  --auto-provision On \
  --subscription <sub-id>
```

⚠️ **Always explain pricing impact before enabling.** Refer to the pricing reference.

---

## Phase 3: Server Discovery & Onboarding — "Where are my servers and how do I bring them in?"

**Goal:** Find all servers (Azure VMs + on-prem via Arc), check their protection status, and onboard any that are missing.

### CLI Commands

**Discover ALL servers across ALL subscriptions (ARG — one query):**
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend os = tostring(properties.osType),
           serverName = name,
           kind = iff(type contains 'hybridcompute', 'On-prem (Arc)', 'Azure VM'),
           powerState = tostring(properties.extended.instanceView.powerState.displayStatus)
  | project serverName, kind, os, location, resourceGroup, subscriptionId, powerState
  | order by kind asc, serverName asc
" --output table
```

**Full posture query — servers + MDE agent + Defender plan (the power query):**
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend os = tostring(properties.osType),
           serverName = name,
           kind = iff(type contains 'hybridcompute', 'On-prem (Arc)', 'Azure VM')
  | join kind=leftouter (
      resources
      | where type =~ 'microsoft.compute/virtualmachines/extensions'
         or type =~ 'microsoft.hybridcompute/machines/extensions'
      | where name in~ ('MDE.Windows', 'MDE.Linux', 'MicrosoftMonitoringAgent', 'AzureMonitorWindowsAgent')
      | extend serverName = tostring(split(id, '/')[8]),
               extName = name,
               extStatus = tostring(properties.provisioningState)
    ) on serverName
  | project serverName, kind, os, location, subscriptionId,
            mdeAgent = iff(extName in~ ('MDE.Windows', 'MDE.Linux'), '✅ Installed', '❌ Missing'),
            agentStatus = coalesce(extStatus, 'N/A')
  | order by mdeAgent asc, kind asc
" --output table
```

**List Arc-connected servers specifically:**
```bash
az connectedmachine list --query "[].{Name:name, OS:osType, Status:status, ResourceGroup:resourceGroup}" --output table
```

### On-Prem Server Onboarding via Azure Arc

**Generate Arc onboarding script:**
```bash
# For a single server
az connectedmachine connect \
  --resource-group <rg-name> \
  --name <server-name> \
  --location <azure-region>

# For bulk onboarding — generate a script
az connectedmachine generate-script \
  --resource-group <rg-name> \
  --location <azure-region> \
  --subscription <sub-id>
```

**Alternative: Direct MDE onboarding (for servers that can't run Arc):**
```bash
# Download MDE onboarding package from security.microsoft.com
# Or use the MDE API to generate onboarding scripts
az rest --method GET \
  --url "https://api.securitycenter.microsoft.com/api/machineactions"
```

### How to Present Results

Show a clear table:
```
🖥️ SERVER INVENTORY
          Total   Protected   Unprotected
Azure VM:   28      22 ✅        6 ❌
On-prem:     0 (customer reports ~20 on-prem servers not yet Arc-connected)

🔧 TO BRING ON-PREM SERVERS IN:
1. Install Azure Arc agent on each server
2. Once Arc-connected, Defender plans auto-apply (if auto-provisioning is ON)
3. MDE agent deploys automatically
```

---

## Phase 4: Recommendations & Vulnerabilities — "Now I see data, what does it mean?"

**Goal:** After enablement and onboarding, data flows into MDC. Help the customer understand and prioritize their recommendations.

### CLI Commands

**Get top unhealthy recommendations across all subscriptions (ARG):**
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/assessments'
  | where tostring(properties.status.code) == 'Unhealthy'
  | extend recommendation = tostring(properties.displayName),
           severity = tostring(properties.metadata.severity),
           category = tostring(properties.metadata.categories[0]),
           resourceId = tostring(properties.resourceDetails.Id)
  | summarize affectedResources = count() by recommendation, severity, category
  | order by severity asc, affectedResources desc
  | take 15
" --output table
```

**Get Secure Score:**
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/securescores'
  | extend scoreName = tostring(properties.displayName),
           currentScore = todouble(properties.score.current),
           maxScore = todouble(properties.score.max),
           percentage = round(todouble(properties.score.percentage) * 100, 1)
  | project subscriptionId, scoreName, currentScore, maxScore, percentage
" --output table
```

**Get score per control (what to fix for biggest impact):**
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/securescores/securescorecontrols'
  | extend controlName = tostring(properties.displayName),
           currentScore = todouble(properties.score.current),
           maxScore = todouble(properties.score.max),
           unhealthyCount = toint(properties.unhealthyResourceCount),
           weight = toint(properties.weight)
  | project subscriptionId, controlName, currentScore, maxScore, unhealthyCount, weight
  | order by weight desc, unhealthyCount desc
  | take 10
" --output table
```

### How to Present Results

```
📊 YOUR SECURITY POSTURE
Secure Score: 34/100 (normal for day 1 — don't panic)

TOP 5 RECOMMENDATIONS (biggest impact first):
# | Recommendation              | Severity | Affected
1 | Enable endpoint protection  | High     | 16 servers
2 | Apply system updates        | High     | 23 servers
3 | Restrict management ports   | High     | 28 servers
4 | Enable disk encryption      | Medium   | 12 servers
5 | Fix vulnerable packages     | Medium   | 31 servers

💡 Start with #1 and #2 — they give the biggest Secure Score lift
   and are the easiest to fix.
```

---

## Phase 5: Posture Summary — "Are my servers secure?"

**Goal:** Give the customer a clear, ongoing view of their security posture. This phase also serves as the **re-run** — the customer can come back and check progress.

### CLI Commands

**Complete posture dashboard (combine everything):**
```bash
# 1. Defender plan status
az graph query -q "
  securityresources
  | where type == 'microsoft.security/pricings'
  | where name in~ ('VirtualMachines', 'CloudPosture', 'SqlServers')
  | project subscriptionId, plan = name, tier = tostring(properties.pricingTier)
" --output table

# 2. Server coverage
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend kind = iff(type contains 'hybridcompute', 'On-prem', 'Azure')
  | summarize total = count() by kind
" --output table

# 3. MDE coverage
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend serverName = name
  | join kind=leftouter (
      resources
      | where type contains 'extensions'
      | where name in~ ('MDE.Windows', 'MDE.Linux')
      | extend serverName = tostring(split(id, '/')[8])
    ) on serverName
  | extend protected = iff(isnotempty(serverName1), 'Protected', 'Exposed')
  | summarize count() by protected
" --output table

# 4. Secure Score
az graph query -q "
  securityresources
  | where type == 'microsoft.security/securescores'
  | extend percentage = round(todouble(properties.score.percentage) * 100, 1)
  | project subscriptionId, percentage
" --output table
```

### How to Present Results

```
🛡️ SERVER SECURITY SUMMARY

         Protected   Exposed
Azure:   22 ✅       6 ❌
On-prem: 12 ✅       4 ❌

Secure Score: 51/100 (up from 34 two weeks ago ✅)

🎯 REMAINING ACTIONS:
1. Onboard remaining 4 on-prem servers via Arc
2. Patch critical vulnerabilities on 3 servers
3. Enable JIT VM access on management ports
4. Schedule monthly re-assessment
```

---

## Handling Common Customer Scenarios

### "I just want to protect my servers"
→ Start Phase 1, fast-track through to Phase 3. Focus on Defender for Servers enablement + MDE agent deployment.

### "I've already enabled Defender but I'm not sure it's working"
→ Jump to Phase 3 (verify MDE agents) then Phase 4 (check if recommendations are flowing).

### "What am I paying for?"
→ Phase 1 (license check) + pricing reference. Show what they have, what's included, what costs extra.

### "How do I get my on-prem servers into the portal?"
→ Phase 3 — Azure Arc onboarding flow.

### "My Secure Score is low — what do I fix first?"
→ Phase 4 — show score controls by weight, prioritize high-impact/low-effort recommendations.

### "I need to report to my CISO that we're secure"
→ Phase 5 — full posture summary with coverage numbers and score trend.
