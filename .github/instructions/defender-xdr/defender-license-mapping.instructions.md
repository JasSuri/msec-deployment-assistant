# Defender XDR License Mapping Reference

## SKU → Defender Product Mapping

### M365 E5 (Full XDR Stack)

| SKU Part Number | License Name | Defender Products Included |
|----------------|--------------|---------------------------|
| `SPE_E5` | Microsoft 365 E5 | MDE P2, MDO P2, MDI, MDA |
| `ENTERPRISEPREMIUM` | Microsoft 365 E5 | MDE P2, MDO P2, MDI, MDA |

### M365 E3 (Limited Defender)

| SKU Part Number | License Name | Defender Products Included |
|----------------|--------------|---------------------------|
| `SPE_E3` | Microsoft 365 E3 | MDE P1 only (basic endpoint) |
| `ENTERPRISEPACK` | Microsoft 365 E3 | MDE P1 only |

### Standalone / Add-on SKUs

| SKU Part Number | License Name | Product |
|----------------|--------------|---------|
| `MDATP` | Defender for Endpoint P2 | MDE P2 (endpoints only, NOT servers) |
| `MDATP_Server` | Defender for Servers P2 | Server protection (→ route to MDC agent) |
| `ATP_ENTERPRISE` | Defender for Office 365 P1 | MDO P1 (Safe Attachments, Safe Links) |
| `THREAT_INTELLIGENCE` | Defender for Office 365 P2 | MDO P2 (+ Threat Explorer, AIR, Attack Sim) |
| `ATA` | Defender for Identity | MDI |
| `M365_E5_SECURITY` | M365 E5 Security add-on | MDE P2, MDO P2, MDI, MDA |

### Product Feature Comparison

| Feature | MDE P1 | MDE P2 | MDO P1 | MDO P2 |
|---------|--------|--------|--------|--------|
| Antivirus / AV | ✅ | ✅ | - | - |
| EDR | ❌ | ✅ | - | - |
| ASR rules | ✅ | ✅ | - | - |
| Automated Investigation | ❌ | ✅ | ❌ | ✅ |
| Threat & Vuln Mgmt | ❌ | ✅ | - | - |
| Safe Attachments | - | - | ✅ | ✅ |
| Safe Links | - | - | ✅ | ✅ |
| Anti-phishing (impersonation) | - | - | ❌ | ✅ |
| Threat Explorer | - | - | ❌ | ✅ |
| Attack Simulation | - | - | ❌ | ✅ |

> ⚠️ **Defender for Servers ≠ Defender for Endpoint.** MDE P2 covers desktops/laptops. For servers, customers need Defender for Servers (MDC) — route to the MDC agent.
