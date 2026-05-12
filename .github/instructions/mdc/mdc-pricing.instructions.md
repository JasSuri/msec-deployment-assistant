# MDC Pricing Reference

Use this reference to calculate costs and present pricing options to customers. Always show **multiple options** so the customer can choose based on budget.

---

## Defender for Cloud Pricing (Pay-as-you-go)

### Defender for Servers

| Plan | Price | What's Included |
|------|-------|-----------------|
| **P1** | ~$5/server/month | EDR, vulnerability assessment, MDE integration |
| **P2** | ~$15/server/month | Everything in P1 + JIT, agentless scanning, FIM, adaptive app controls |
| **P1 with M365 E5** | **$0** (included) | Same as P1 — already entitled, just enable it |
| **P2 upgrade from E5 P1** | ~$10/server/month delta | P2 capabilities on top of included P1 |

### Defender CSPM

| Plan | Price | What's Included |
|------|-------|-----------------|
| **Free (Foundational CSPM)** | $0 | Secure Score, basic recommendations, Azure security benchmark |
| **Paid (Defender CSPM)** | ~$5/server/month (billed per billable resource) | Attack path analysis, cloud security explorer, governance, agentless scanning, data-aware posture |

### Other Defender for Cloud Plans

| Plan | Price | Billable Unit |
|------|-------|---------------|
| Defender for Storage | ~$10/storage account/month | Per storage account |
| Defender for SQL (Azure) | ~$15/instance/month | Per SQL instance |
| Defender for SQL on VMs | ~$15/instance/month | Per SQL VM |
| Defender for App Service | ~$15/instance/month | Per App Service instance |
| Defender for Key Vault | ~$0.02/10K transactions | Per transaction |
| Defender for ARM | ~$4/subscription/month | Per subscription |
| Defender for DNS | ~$0.70/million queries | Per DNS query |
| Defender for Containers | ~$7/vCore/month | Per container vCore |
| Defender for APIs | ~$0.20/1K API calls | Per API call |

---

## How to Calculate Customer Pricing

### Step 1: Identify Workloads

Run this ARG query to count billable resources:

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
      type contains 'sql', 'SQL Instances',
      type contains 'storage', 'Storage Accounts',
      type contains 'web/sites', 'App Services',
      type contains 'keyvault', 'Key Vaults',
      type contains 'containerservice', 'AKS Clusters',
      'Other'
    )
  | summarize count() by resourceType
  | order by count_ desc
" --output table
```

### Step 2: Check Existing Entitlements

If the customer has M365 E5/E5 Security:
- Subtract the P1 cost (it's free)
- Only quote the P2 upgrade delta ($10/server/month)

### Step 3: Present Options

Always show 3 options:

```
💰 PRICING OPTIONS (based on 47 servers, 2 SQL instances)

┌─────────────────────────────────────────────────────┐
│ OPTION A: Free tier + E5 entitlement                │
│                                                     │
│ Foundational CSPM:     $0                           │
│ Servers P1 (via E5):   $0 (already included)        │
│                                                     │
│ Monthly total:         $0/month                     │
│                                                     │
│ ⚠️ You get: basic recommendations, EDR, vuln scan   │
│ ❌ You miss: JIT, agentless scanning, attack paths   │
├─────────────────────────────────────────────────────┤
│ OPTION B: Servers P2 upgrade (recommended ✅)       │
│                                                     │
│ Servers P2 delta:    47 × $10/mo = $470/mo          │
│ Defender for SQL:     2 × $15/mo = $30/mo           │
│                                                     │
│ Monthly total:       ~$500/month                    │
│                                                     │
│ ✅ Full server protection + agentless scanning       │
├─────────────────────────────────────────────────────┤
│ OPTION C: Full stack (P2 + paid CSPM)               │
│                                                     │
│ Servers P2 delta:    47 × $10/mo = $470/mo          │
│ Defender CSPM:       47 × $5/mo  = $235/mo          │
│ Defender for SQL:     2 × $15/mo = $30/mo           │
│                                                     │
│ Monthly total:       ~$735/month                    │
│                                                     │
│ ✅ Everything: attack paths, security explorer,      │
│    governance rules, data-aware posture              │
└─────────────────────────────────────────────────────┘
```

---

## Pricing Conversation Tips

1. **Always lead with the free entitlement.** "Good news — you're already entitled to P1 at no cost with your E5 license."

2. **Frame P2 as a delta, not full price.** Say "$10/server upgrade" not "$15/server" — they're already paying for E5.

3. **Anchor to risk, not features.** "Without P2, you have no way to restrict management ports on your servers (JIT). That's your #1 attack vector."

4. **Compare to breach cost.** "$500/month for 47 servers is ~$10/server. A single ransomware incident costs $100K+ on average."

5. **Start small.** "Enable P2 on your 10 admin/critical servers first. That's $100/month. Expand after you see value."

---

## Common Pricing Questions

### "Why do I need to pay more if I already have E5?"
E5 covers **endpoints** (desktops/laptops) and gives you **server P1**. Server P2 adds cloud-native protections (JIT, agentless scanning) that don't exist in the endpoint product.

### "Can I mix P1 and P2 across servers?"
Yes. Enable P2 on critical/production servers and keep P1 on dev/test. You choose per-subscription or per-resource (with sub-plan configuration).

### "Is there a discount for annual commitment?"
Defender for Cloud is pay-as-you-go by default. For EA/MCA customers, pricing may be negotiable through their Microsoft account team.

### "What if I turn off servers at night?"
You're only billed when the server is running. Deallocated VMs are not billed for Defender for Servers.
