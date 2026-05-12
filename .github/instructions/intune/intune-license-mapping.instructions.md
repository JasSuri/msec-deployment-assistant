# Intune License Mapping Reference

## SKU → Intune Entitlement Mapping

### Licenses That Include Intune (Full MDM + MAM)

| SKU Part Number | License Name | Intune Entitlement |
|----------------|--------------|-------------------|
| `SPE_E5` | Microsoft 365 E5 | Intune Plan 1 |
| `SPE_E3` | Microsoft 365 E3 | Intune Plan 1 |
| `ENTERPRISEPREMIUM` | Microsoft 365 E5 | Intune Plan 1 |
| `ENTERPRISEPACK` | Microsoft 365 E3 | Intune Plan 1 |
| `EMSPREMIUM` | EMS E5 | Intune Plan 1 |
| `EMSPREMIUM_E3` | EMS E3 | Intune Plan 1 |
| `INTUNE_A` | Intune Plan 1 standalone | Intune Plan 1 |
| `Microsoft_365_Business_Premium` | M365 Business Premium | Intune Plan 1 |

### Intune Add-ons

| SKU Part Number | License Name | What It Adds |
|----------------|--------------|-------------|
| `INTUNE_P2` | Intune Plan 2 | Advanced endpoint management (specialty devices, Cloud PKI) |
| `INTUNE_SUITE` | Intune Suite | Remote Help, Tunnel, ADEM, privilege management, firmware OTA |

### What's Included in Intune Plan 1

| Feature | Plan 1 | Plan 2 | Suite |
|---------|--------|--------|-------|
| MDM (device enrollment & management) | ✅ | ✅ | ✅ |
| MAM (app protection without enrollment) | ✅ | ✅ | ✅ |
| Compliance policies | ✅ | ✅ | ✅ |
| Configuration profiles | ✅ | ✅ | ✅ |
| Autopilot | ✅ | ✅ | ✅ |
| Windows Update management | ✅ | ✅ | ✅ |
| Endpoint analytics | ✅ | ✅ | ✅ |
| Cloud PKI | ❌ | ✅ | ✅ |
| Specialty devices (AR/VR, kiosk) | ❌ | ✅ | ✅ |
| Remote Help | ❌ | ❌ | ✅ |
| Tunnel for MAM | ❌ | ❌ | ✅ |
| Endpoint Privilege Management | ❌ | ❌ | ✅ |
| Advanced device analytics | ❌ | ❌ | ✅ |
