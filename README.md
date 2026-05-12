# Security Deployment Demos

AI-assisted deployment workshops for Microsoft Security products using GitHub Copilot.

## Overview

This repository provides context files (agents, skills, instructions) that give GitHub Copilot deep knowledge of Microsoft Security products. When loaded, Copilot can guide IT admins through end-to-end deployment workflows — from license assessment to enablement and validation.

## Supported Products

| Product | Status | Folder |
|---------|--------|--------|
| Microsoft Defender for Cloud (MDC) | ✅ Ready | `skills/mdc/` |
| Microsoft Entra (Identity & Access) | ✅ Ready | `skills/entra/` |
| Microsoft Intune (Endpoint Management) | ✅ Ready | `skills/intune/` |
| Microsoft Defender XDR (Endpoint, Email, Identity, Cloud Apps) | ✅ Ready | `skills/defender-xdr/` |
| Microsoft Purview (Data Security & Compliance) | ✅ Ready | `skills/purview/` |
| Microsoft Sentinel (SIEM/SOAR) | 🔜 Planned | `skills/sentinel/` |

## How It Works

1. **Open this folder in VS Code** with GitHub Copilot enabled
2. **Copilot reads the context files** (`.github/copilot-instructions.md` → agents → skills → instructions)
3. **Ask Copilot**: _"I want to use MDC to protect my cloud environments, help me plan and deploy"_
4. **Copilot walks you through** a structured workshop: license check → enablement → server discovery → recommendations → posture summary

## Prerequisites

- GitHub Copilot (Chat) in VS Code
- Azure CLI (`az`) installed and authenticated (`az login`)
- Azure subscription with target resources deployed (see [Lab Setup](#lab-setup))
- Appropriate Azure permissions (Owner or Security Admin on target subscriptions)

## Lab Setup

To create a demo environment with resources that MDC can protect, deploy the official MDC lab ARM template:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Security-Center%2Fmaster%2FLabs%2FFiles%2Flabdeploy.json)

This deploys: Windows VM, Linux VM, AKS cluster, SQL Server, Key Vault, Storage Account, App Service, and more. Takes ~10 minutes.

Source: [MDC Lab Module 1](https://github.com/Azure/Microsoft-Defender-for-Cloud/blob/main/Labs/Modules/Module-1-Preparing-the-Environment.md)

## Folder Structure

```
security-deployment-demos/
├── .github/
│   ├── copilot-instructions.md         # Main agent — routes to product skills
│   └── instructions/
│       └── mdc/                        # MDC-specific instruction files
│           ├── mdc-workshop-phases.instructions.md
│           ├── mdc-license-mapping.instructions.md
│           ├── mdc-pricing.instructions.md
│           └── mdc-arg-queries.instructions.md
├── agents/
│   └── mdc-deployment.md              # MDC deployment agent persona
├── skills/
│   └── mdc/
│       └── SKILL.md                   # MDC skill definition
├── demo-scripts/
│   └── mdc-demo-walkthrough.md        # Demo recording script
└── README.md                          # This file
```

## Adding a New Product

To add support for a new product (e.g., Entra):

1. Create `skills/entra/SKILL.md` — skill definition
2. Create `agents/entra-deployment.md` — agent persona
3. Create `.github/instructions/entra/` — detailed instruction files
4. Update `.github/copilot-instructions.md` to route to the new product
5. Update this README

## Recording a Demo

See `demo-scripts/mdc-demo-walkthrough.md` for the MDC demo talk track and step-by-step recording guide.
