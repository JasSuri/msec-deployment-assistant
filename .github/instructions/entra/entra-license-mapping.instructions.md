# Entra License Mapping Reference

## SKU → Entra Entitlement Mapping

### Licenses That Include Entra ID P2 (full feature set)

| SKU Part Number | License Name | Entra Entitlement |
|----------------|--------------|-------------------|
| `SPE_E5` | Microsoft 365 E5 | Entra ID P2 |
| `ENTERPRISEPREMIUM` | Microsoft 365 E5 | Entra ID P2 |
| `EMSPREMIUM` | EMS E5 | Entra ID P2 |
| `M365_E5_SECURITY` | M365 E5 Security | Entra ID P2 |
| `AAD_PREMIUM_P2` | Entra ID P2 standalone | Entra ID P2 |

### Licenses That Include Entra ID P1 (Conditional Access, MFA, but no Identity Protection)

| SKU Part Number | License Name | Entra Entitlement |
|----------------|--------------|-------------------|
| `SPE_E3` | Microsoft 365 E3 | Entra ID P1 |
| `ENTERPRISEPACK` | Microsoft 365 E3 | Entra ID P1 |
| `EMSPREMIUM_E3` | EMS E3 | Entra ID P1 |
| `AAD_PREMIUM` | Entra ID P1 standalone | Entra ID P1 |

### P1 vs P2 Feature Comparison

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| SSO (unlimited apps) | ✅ | ✅ | ✅ |
| MFA | ✅ (Security Defaults) | ✅ (Conditional Access) | ✅ |
| Conditional Access | ❌ | ✅ | ✅ |
| Self-service password reset | ❌ | ✅ | ✅ |
| Identity Protection (risk policies) | ❌ | ❌ | ✅ |
| Privileged Identity Mgmt (PIM) | ❌ | ❌ | ✅ |
| Access Reviews | ❌ | ❌ | ✅ |
| Entitlement Management | ❌ | ❌ | ✅ |
