# Purview License Mapping Reference

## SKU → Purview Feature Mapping

### M365 E5 (Full Purview Suite)

| SKU Part Number | License Name | Purview Features |
|----------------|--------------|-----------------|
| `SPE_E5` | Microsoft 365 E5 | All Purview features |
| `ENTERPRISEPREMIUM` | Microsoft 365 E5 | All Purview features |
| `INFORMATION_PROTECTION_COMPLIANCE` | M365 E5 Compliance add-on | All Purview features (add to E3) |
| `M365_E5_COMPLIANCE` | M365 E5 Compliance | All Purview features |

### M365 E3 (Basic Purview)

| SKU Part Number | License Name | Purview Features |
|----------------|--------------|-----------------|
| `SPE_E3` | Microsoft 365 E3 | Basic labels, DLP (Exchange/SPO), retention, eDiscovery Standard |
| `ENTERPRISEPACK` | Microsoft 365 E3 | Same as above |

### Feature Comparison: E3 vs E5

| Feature | E3 | E5 / E5 Compliance |
|---------|----|--------------------|
| Sensitivity labels (manual) | ✅ | ✅ |
| Auto-labeling (client-side) | ✅ | ✅ |
| Auto-labeling (service-side) | ❌ | ✅ |
| DLP (Exchange, SPO, OneDrive) | ✅ | ✅ |
| DLP (Teams) | ❌ | ✅ |
| Endpoint DLP | ❌ | ✅ |
| Retention policies & labels | ✅ | ✅ |
| Records management | ❌ | ✅ |
| eDiscovery Standard | ✅ | ✅ |
| eDiscovery Premium | ❌ | ✅ |
| Insider Risk Management | ❌ | ✅ |
| Communication Compliance | ❌ | ✅ |
| Information Barriers | ❌ | ✅ |
| Customer Key | ❌ | ✅ |
| Privileged Access Management | ❌ | ✅ |
| Compliance Manager (assessments) | Basic | Premium (all templates) |

### Standalone Add-ons

| SKU | What It Adds |
|-----|-------------|
| `INFORMATION_PROTECTION_COMPLIANCE` | Full E5 Compliance to E3 |
| `COMPLIANCE_MANAGER_PREMIUM_ASSESSMENT` | Additional compliance templates |
| `MIP_S_EXCHANGE` | Information Protection for Exchange |
