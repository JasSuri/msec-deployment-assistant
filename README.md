# Security Deployment Demos

AI-assisted deployment workshops for Microsoft Security products using GitHub Copilot.

## Overview

This repository provides context files (agents, skills, instructions) that give GitHub Copilot deep knowledge of Microsoft Security products. When loaded, Copilot can guide IT admins through end-to-end deployment workflows ÔÇö from license assessment to enablement and validation.

## GitHub Copilot CLI plugin

This repository can be installed as a GitHub Copilot CLI plugin.

### Direct install

```shell
copilot plugin install OWNER/REPO
```

Or from a Git URL:

```shell
copilot plugin install https://github.com/OWNER/REPO.git
```

### Marketplace mode

The repo also includes `.github/plugin/marketplace.json`, so it can be registered as a plugin marketplace as well as installed directly as a plugin.

### Packaging model

- `products/` is the canonical source for structured product packs.
- `agents/` contains plugin-compatible custom agents.
- `skills/` contains plugin-compatible skills.
- `.github/copilot-instructions.md` keeps direct repository usage deterministic.

## Supported Products

| Product | Status | Folder |
|---------|--------|--------|
| Microsoft Defender for Cloud (MDC) | Ô£à Ready | `skills/mdc/` |
| Microsoft Entra (Identity & Access) | Ô£à Ready | `skills/entra/` |
| Microsoft Intune (Endpoint Management) | Ô£à Ready | `skills/intune/` |
| Microsoft Defender XDR (Endpoint, Email, Identity, Cloud Apps) | Ô£à Ready | `skills/defender-xdr/` |
| Microsoft Purview (Data Security & Compliance) | Ô£à Ready | `skills/purview/` |
| Microsoft Sentinel (SIEM/SOAR) | ­ƒö£ Planned | `skills/sentinel/` |

## How It Works

1. **Open this folder in VS Code** with GitHub Copilot enabled
2. **Copilot reads the context files** (`.github/copilot-instructions.md` ÔåÆ agents ÔåÆ skills ÔåÆ instructions)
3. **Ask Copilot**: _"I want to use MDC to protect my cloud environments, help me plan and deploy"_
4. **Copilot walks you through** a structured workshop: license check ÔåÆ enablement ÔåÆ server discovery ÔåÆ recommendations ÔåÆ posture summary

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
Ôö£ÔöÇÔöÇ .github/
Ôöé   Ôö£ÔöÇÔöÇ copilot-instructions.md         # Main agent ÔÇö routes to product skills
Ôöé   ÔööÔöÇÔöÇ instructions/
Ôöé       ÔööÔöÇÔöÇ mdc/                        # MDC-specific instruction files
Ôöé           Ôö£ÔöÇÔöÇ mdc-workshop-phases.instructions.md
Ôöé           Ôö£ÔöÇÔöÇ mdc-license-mapping.instructions.md
Ôöé           Ôö£ÔöÇÔöÇ mdc-pricing.instructions.md
Ôöé           ÔööÔöÇÔöÇ mdc-arg-queries.instructions.md
Ôö£ÔöÇÔöÇ agents/
Ôöé   ÔööÔöÇÔöÇ mdc-deployment.md              # MDC deployment agent persona
Ôö£ÔöÇÔöÇ skills/
Ôöé   ÔööÔöÇÔöÇ mdc/
Ôöé       ÔööÔöÇÔöÇ SKILL.md                   # MDC skill definition
Ôö£ÔöÇÔöÇ demo-scripts/
Ôöé   ÔööÔöÇÔöÇ mdc-demo-walkthrough.md        # Demo recording script
ÔööÔöÇÔöÇ README.md                          # This file
```

## Adding a New Product

To add support for a new product (e.g., Entra):

1. Create `products/<product>/` from `_template/` when you want a structured source pack.
2. Add or update `skills/<product>/SKILL.md` so the plugin can load the skill directly.
3. Add `agents/<product>-deployment.agent.md` so the plugin exposes a product-specific custom agent.
4. Add or update `.github/instructions/<product>/` for any product-specific supporting guidance.
5. Update `.github/copilot-instructions.md` routing if needed.
6. Update this README.

## Recording a Demo

See `demo-scripts/mdc-demo-walkthrough.md` for the MDC demo talk track and step-by-step recording guide.
