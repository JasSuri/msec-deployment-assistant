# MDC License Mapping Reference

Use this reference to map Microsoft 365 and Azure SKU part numbers (from `az rest --url .../subscribedSkus`) to Defender for Cloud entitlements.

---

## SKU → Defender Entitlement Mapping

### Licenses That INCLUDE Defender for Servers P1 (no extra cost)

| SKU Part Number | License Name | Defender Entitlement |
|----------------|--------------|---------------------|
| `ENTERPRISEPREMIUM` | Microsoft 365 E5 | Defender for Servers P1 |
| `SPE_E5` | Microsoft 365 E5 | Defender for Servers P1 |
| `M365_E5_SECURITY` | Microsoft 365 E5 Security | Defender for Servers P1 |
| `IDENTITY_THREAT_PROTECTION` | Microsoft 365 E5 Security | Defender for Servers P1 |
| `Microsoft_365_E5_Security` | Microsoft 365 E5 Security add-on | Defender for Servers P1 |

### Licenses That INCLUDE Defender for Servers P2

| SKU Part Number | License Name | Defender Entitlement |
|----------------|--------------|---------------------|
| `MDATP_Server` | Defender for Servers P2 (standalone) | Defender for Servers P2 |
| `DEFENDER_FOR_SERVERS_P2` | Defender for Servers Plan 2 | Defender for Servers P2 |

### Licenses That Include Defender CSPM (Paid)

| SKU Part Number | License Name | Defender Entitlement |
|----------------|--------------|---------------------|
| `DEFENDER_CSPM` | Defender CSPM | Full CSPM (attack paths, governance) |

### Licenses That Do NOT Include Any Server Protection

These are common SKUs customers think cover servers but DON'T:

| SKU Part Number | License Name | What It Actually Covers |
|----------------|--------------|------------------------|
| `MDATP` | Defender for Endpoint P2 | **Endpoints (desktops/laptops) only** — NOT servers |
| `MDE_SMB` | Defender for Business | Endpoints for SMB — NOT servers |
| `ATP_ENTERPRISE` | Defender for Office 365 P1 | Email/Office apps only |
| `THREAT_INTELLIGENCE` | Defender for Office 365 P2 | Email/Office apps only |
| `ATA` | Defender for Identity | Active Directory only |
| `ENTERPRISEPACK` | Microsoft 365 E3 | No Defender for Servers at all |
| `SPE_E3` | Microsoft 365 E3 | No Defender for Servers at all |

---

## Critical Licensing Distinction: Server vs Device

### ⚠️ This is the #1 confusion point for SMC customers

**Defender for Endpoint (MDE)** = per-user/device license for **desktops, laptops, phones**
**Defender for Servers** = per-server license for **servers (Azure VMs, Arc, on-prem)**

They are **separate products with separate licensing:**

| Feature | Defender for Endpoint | Defender for Servers P1 | Defender for Servers P2 |
|---------|----------------------|------------------------|------------------------|
| Target | Desktops/laptops | Servers | Servers |
| Licensing | Per user | Per server | Per server |
| EDR | ✅ | ✅ | ✅ |
| Vulnerability mgmt | ✅ | ✅ | ✅ |
| JIT VM access | ❌ | ❌ | ✅ |
| Agentless scanning | ❌ | ❌ | ✅ |
| Adaptive app controls | ❌ | ❌ | ✅ |
| File integrity monitoring | ❌ | ❌ | ✅ |
| Docker host hardening | ❌ | ❌ | ✅ |
| Network map | ❌ | ❌ | ✅ |

---

## P1 vs P2 — What the Customer Misses on P1

When a customer has P1 included with E5 but hasn't purchased P2, explain:

**P1 gives you:**
- MDE agent on servers (antivirus + EDR)
- Vulnerability assessment
- Integration with Defender XDR

**P2 additionally gives you:**
- Just-in-time (JIT) VM access — reduce attack surface on management ports
- Agentless vulnerability scanning — no agent needed, scans VM disks directly
- Adaptive application controls — allowlist approved applications
- File integrity monitoring (FIM) — detect unauthorized file changes
- Docker host hardening — CIS benchmarks for container hosts
- Network map — visualize network topology and traffic

**Upgrade cost:** P2 is an additional ~$10/server/month on top of P1 entitlement.

---

## How to Present License Findings

Always structure the output as:

```
📦 YOUR LICENSES
 ✅ [What they have] — [what it includes]
 ❌ [What they don't have] — [what they're missing]
 ⚠️ [Common confusion] — [clarification]

💡 RECOMMENDATION
 [What they should consider based on their workloads]
 [Cost estimate based on server count]
```
