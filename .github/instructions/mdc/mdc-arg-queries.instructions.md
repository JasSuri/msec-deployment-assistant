# MDC Azure Resource Graph (ARG) Query Reference

All queries use `az graph query -q "..."` and work across all customer subscriptions in a single call.

---

## Quick Reference — Copy-Paste Queries

### 1. List All Defender Plans and Status (All Subs)
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/pricings'
  | extend plan = name, tier = tostring(properties.pricingTier), subPlan = tostring(properties.subPlan)
  | project subscriptionId, plan, tier, subPlan
  | order by subscriptionId, plan
" --output table
```

### 2. List All Servers (Azure VMs + Arc)
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend os = tostring(properties.osType),
           kind = iff(type contains 'hybridcompute', 'Arc (on-prem)', 'Azure VM'),
           powerState = tostring(properties.extended.instanceView.powerState.displayStatus)
  | project name, kind, os, location, resourceGroup, subscriptionId, powerState
  | order by kind, name
" --output table
```

### 3. Server Count by Type and OS
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend os = tostring(properties.osType),
           kind = iff(type contains 'hybridcompute', 'Arc', 'Azure')
  | summarize count() by kind, os
" --output table
```

### 4. MDE Agent Status Per Server
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines/extensions', 'microsoft.hybridcompute/machines/extensions')
  | where name in~ ('MDE.Windows', 'MDE.Linux')
  | extend serverName = tostring(split(id, '/')[8]),
           agentType = name,
           status = tostring(properties.provisioningState)
  | project serverName, agentType, status
  | order by status, serverName
" --output table
```

### 5. Servers WITHOUT MDE Agent (unprotected)
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend serverName = name
  | join kind=anti (
      resources
      | where type in~ ('microsoft.compute/virtualmachines/extensions', 'microsoft.hybridcompute/machines/extensions')
      | where name in~ ('MDE.Windows', 'MDE.Linux')
      | extend serverName = tostring(split(id, '/')[8])
    ) on serverName
  | project serverName, type, os = tostring(properties.osType), location, subscriptionId
  | order by serverName
" --output table
```

### 6. Full Server Posture (the power query)
```bash
az graph query -q "
  resources
  | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
  | extend serverName = name,
           os = tostring(properties.osType),
           kind = iff(type contains 'hybridcompute', 'Arc', 'Azure')
  | join kind=leftouter (
      resources
      | where type in~ ('microsoft.compute/virtualmachines/extensions', 'microsoft.hybridcompute/machines/extensions')
      | where name in~ ('MDE.Windows', 'MDE.Linux')
      | extend serverName = tostring(split(id, '/')[8]),
               mdeStatus = tostring(properties.provisioningState)
    ) on serverName
  | project serverName, kind, os, location, subscriptionId,
            mdeAgent = iff(isnotempty(mdeStatus), 'Installed', 'Missing'),
            mdeHealth = coalesce(mdeStatus, 'N/A')
  | order by mdeAgent asc, kind asc
" --output table
```

### 7. Secure Score Per Subscription
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/securescores'
  | extend currentScore = todouble(properties.score.current),
           maxScore = todouble(properties.score.max),
           pct = round(todouble(properties.score.percentage) * 100, 1)
  | project subscriptionId, currentScore, maxScore, pct
" --output table
```

### 8. Top Unhealthy Recommendations
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/assessments'
  | where tostring(properties.status.code) == 'Unhealthy'
  | extend recommendation = tostring(properties.displayName),
           severity = tostring(properties.metadata.severity),
           category = tostring(properties.metadata.categories[0])
  | summarize affected = count() by recommendation, severity, category
  | order by severity asc, affected desc
  | take 10
" --output table
```

### 9. Secure Score Controls (what to fix for biggest impact)
```bash
az graph query -q "
  securityresources
  | where type == 'microsoft.security/securescores/securescorecontrols'
  | extend control = tostring(properties.displayName),
           current = todouble(properties.score.current),
           max = todouble(properties.score.max),
           unhealthy = toint(properties.unhealthyResourceCount),
           weight = toint(properties.weight)
  | project subscriptionId, control, current, max, unhealthy, weight
  | order by weight desc
  | take 10
" --output table
```

### 10. Count All Billable Resources (for pricing)
```bash
az graph query -q "
  resources
  | where type in~ (
      'microsoft.compute/virtualmachines',
      'microsoft.hybridcompute/machines',
      'microsoft.sql/servers/databases',
      'microsoft.storage/storageaccounts',
      'microsoft.web/sites',
      'microsoft.keyvault/vaults',
      'microsoft.containerservice/managedclusters'
    )
  | extend resourceType = case(
      type contains 'virtualmachines', 'VMs',
      type contains 'hybridcompute', 'Arc Servers',
      type contains 'sql', 'SQL',
      type contains 'storage', 'Storage',
      type contains 'web/sites', 'App Services',
      type contains 'keyvault', 'Key Vaults',
      type contains 'containerservice', 'AKS',
      'Other'
    )
  | summarize count() by resourceType
  | order by count_ desc
" --output table
```

---

## ARG Tips

- **All queries run across all accessible subscriptions** — no need to specify `--subscription`
- **Results are limited to 1000 rows by default** — add `--first 5000` for larger tenants
- **Use `--output json`** for programmatic processing, `--output table` for human-readable
- **Queries are read-only** — ARG cannot modify resources
- **If the customer gets errors**, check they have `Reader` role on the subscriptions they want to query
