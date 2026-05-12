# MDC Preflight — read-only detection commands

The agent runs these **before asking the user** in Pillar 2. All commands here are **read-only**. Any state-changing command lives in `playbook.md`.

Conventions:
- `{{subId}}` — Azure subscription id
- `{{userObjectId}}` — current signed-in user object id (`az ad signed-in-user show --query id -o tsv`)
- `{{mgId}}` — management group id

---

## 0. Identity baseline

```bash
# Who am I?
az account show --query "{tenant:tenantId, user:user.name, sub:id}" -o table

# My object id (used for role lookups)
az ad signed-in-user show --query id -o tsv
```

## 1. Subscription & MG topology  *(maps to validation.yaml → p2.3)*

```bash
az account list --query "[].{name:name, id:id, state:state, tenantId:tenantId}" -o table
az account management-group list -o table
```

## 2. Resource provider state  *(p2.5)*

```bash
az provider show -n Microsoft.Security --query registrationState -o tsv
```

If output != `Registered`, surface remediation (do NOT auto-run without consent):
```bash
az provider register --namespace Microsoft.Security
```

## 3. Defender plan state per subscription  *(p2.1)*

```bash
# All plans + tier + sub-plan
az security pricing list --subscription {{subId}} \
  --query "value[].{plan:name, tier:pricingTier, subPlan:subPlan, freeTrialRemainingTime:freeTrialRemainingTime}" \
  -o table
```

Cross-reference each row against `license-matrix.yaml → workload_plans[].plan_id`.

## 4. MDC settings & auto-provisioning  *(p2.6)*

```bash
az security setting list --subscription {{subId}} -o table
az security auto-provisioning-setting list --subscription {{subId}} -o table
az security workspace-setting list --subscription {{subId}} -o table
```

## 5. Log Analytics workspaces  *(p2.6)*

```bash
az monitor log-analytics workspace list -o table
```

## 6. Role assessment for the signed-in user  *(p2.7)*

```bash
# RBAC roles at subscription scope
az role assignment list \
  --assignee {{userObjectId}} \
  --scope /subscriptions/{{subId}} \
  --query "[].{role:roleDefinitionName, scope:scope}" -o table

# RBAC roles at MG scope (if applicable)
az role assignment list \
  --assignee {{userObjectId}} \
  --scope /providers/Microsoft.Management/managementGroups/{{mgId}} \
  --query "[].{role:roleDefinitionName, scope:scope}" -o table
```

Microsoft Graph — directory roles (Security Administrator, Global Reader, etc.):
```http
GET https://graph.microsoft.com/v1.0/me/memberOf/microsoft.graph.directoryRole
```

Decision matrix:
| Has at least one of… | Can do |
|---|---|
| Owner / Security Admin (sub) | Enable/disable Defender plans |
| Contributor (resource) | Apply remediation |
| Security Reader | View only |
| *None of the above* | **BLOCKED** — request via PIM |

## 7. Tenant licence inventory  *(informational, p2.1)*

```http
GET https://graph.microsoft.com/v1.0/subscribedSkus?$select=skuPartNumber,consumedUnits,prepaidUnits
```

> Note: MDC is per-Azure-resource, not per-user. This call is for context on adjacent products (M365 E5 → MDE/Purview/Defender XDR redirect signals).

## 8. Multi-cloud connector state  *(p2.4)*

```bash
# Existing security connectors (AWS / GCP / GitHub / Azure DevOps)
az security security-connector list --subscription {{subId}} -o table
```

## 9. Azure Arc machines  *(p2.4 hybrid branch)*

```bash
az connectedmachine list --query "[].{name:name, rg:resourceGroup, status:status, os:osName}" -o table
```

## 10. Existing security policy assignments  *(p2.5 conflict check)*

```bash
az policy assignment list --query "[?contains(displayName, 'Security') || contains(displayName, 'Defender')].{name:displayName, scope:scope}" -o table
```

---

## Output format the agent should produce after preflight

```
PREFLIGHT REPORT — sub: <name> (<id>)
─────────────────────────────────────
Provider Microsoft.Security : Registered ✅
Defender plans enabled      : VirtualMachines (P2) ✅, StorageAccounts (V2) ✅, Containers ❌
Workspace                   : la-prod-eastus ✅
Your roles                  : Contributor (sub) ⚠️  — needs Security Admin to enable plans
Connectors                  : 0 AWS / 0 GCP / 0 DevOps
Arc machines                : 0
Conflicting policies        : none

GAPS DETECTED:
  ⚠️ Role gap     → request Security Admin via PIM (mitigation in validation.yaml b.no_security_admin)
  ⚠️ Containers   → not enabled; required for the AKS protection goal from p1.2
  ⚠️ Defender CSPM→ disabled; required for attack-path goal from p1.2

Next step: ask user to confirm role request + plan selection before Pillar 3.
```
