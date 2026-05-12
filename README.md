# Microsoft Security Deployment Assistant

AI-assisted deployment workshops for Microsoft Security products using GitHub Copilot.

The assistant helps an IT admin go from *"I want to use product X"* to a successfully deployed solution — **without fear** — by following a 4-pillar flow: **Understand → Validate → Guide → Verify**.

## Architecture at a glance

```
👤 IT Admin → Copilot Chat → 🔀 router → product agent → Pillars 1-4 → ☁️ Azure
                                              │
                                              └── reads spec/<product>/* (source of truth)
                                                  reads agents/, skills/, .github/instructions (runtime)
```

Two intentional layers:

| Layer | Purpose | Lives in |
|---|---|---|
| **Spec** | Machine-readable source of truth — questions, gates, detect commands, licence map, verify steps, rollback. Versioned like code. | [`AGENTS.md`](AGENTS.md), [`spec/`](spec/) |
| **Runtime** | What Copilot loads in a chat session — agent personas, instruction blobs, skill registrations. | [`agents/`](agents/), [`skills/`](skills/), [`.github/`](.github/) |

📐 **Diagrams**: [`docs/architecture-diagram.md`](docs/architecture-diagram.md) — components, per-turn flow, verify sequence
📖 **Architecture deep-dive**: [`docs/architecture.md`](docs/architecture.md)
🔌 **Wiring into Copilot / Foundry / Studio**: [`docs/integration.md`](docs/integration.md)
📜 **Orchestrator contract**: [`AGENTS.md`](AGENTS.md)

## Supported products

| Product | Spec | Runtime | Status |
|---|---|---|---|
| Microsoft Defender for Cloud (MDC) | [`spec/mdc/`](spec/mdc/) | [`agents/mdc-deployment.md`](agents/mdc-deployment.md), [`skills/mdc/`](skills/mdc/), [`.github/instructions/mdc/`](.github/instructions/mdc/) | ✅ v1 demo |
| Microsoft Entra (Identity & Access) | ⏳ planned | [`agents/entra-deployment.md`](agents/entra-deployment.md), [`skills/entra/`](skills/entra/) | Runtime ready, spec pending |
| Microsoft Intune (Endpoint Management) | ⏳ planned | [`agents/intune-deployment.md`](agents/intune-deployment.md), [`skills/intune/`](skills/intune/) | Runtime ready, spec pending |
| Microsoft Defender XDR | ⏳ planned | [`agents/defender-xdr-deployment.md`](agents/defender-xdr-deployment.md), [`skills/defender-xdr/`](skills/defender-xdr/) | Runtime ready, spec pending |
| Microsoft Purview | ⏳ planned | [`agents/purview-deployment.md`](agents/purview-deployment.md), [`skills/purview/`](skills/purview/) | Runtime ready, spec pending |
| Microsoft Sentinel | ⏳ planned | — | Planned |

## How it works

1. Open this folder in VS Code with GitHub Copilot enabled.
2. Copilot reads the runtime context: [`.github/copilot-instructions.md`](.github/copilot-instructions.md) → [`agents/orchestrator.md`](agents/orchestrator.md) → [`agents/<product>-deployment.md`](agents/) → [`skills/<product>/SKILL.md`](skills/) → [`.github/instructions/<product>/`](.github/instructions/).
3. Ask Copilot: *"I want to use MDC to protect my cloud environments, help me plan and deploy."*
4. Copilot walks you through the 4 pillars:
   - **Pillar 1 (Understand)** — one open question, catches misconceptions and wrong-product redirects.
   - **Pillar 2 (Validate)** — detects state via `az` / Graph; cross-references licences; only asks what it can't detect.
   - **Pillar 3 (Guide)** — phased deployment, smallest blast radius first, explicit confirm before any change.
   - **Pillar 4 (Verify)** — L1 (sync acknowledge) → L2 (poll for state) → L3 (deferred functional probe). Diagnostics + rollback on failure.

## Prerequisites

- GitHub Copilot (Chat) in VS Code
- Azure CLI (`az`) installed and authenticated (`az login`)
- `az graph` extension (`az extension add --name resource-graph`)
- Azure subscription with target resources deployed (see [Lab Setup](#lab-setup))
- Appropriate Azure permissions (Owner or Security Admin on target subscriptions)

## Lab setup

Deploy the official MDC lab ARM template to get a demo environment with resources MDC can protect:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Security-Center%2Fmaster%2FLabs%2FFiles%2Flabdeploy.json)

Deploys: Windows VM, Linux VM, AKS cluster, SQL Server, Key Vault, Storage Account, App Service, and more (~10 minutes).

Source: [MDC Lab Module 1](https://github.com/Azure/Microsoft-Defender-for-Cloud/blob/main/Labs/Modules/Module-1-Preparing-the-Environment.md)

## Folder structure

```
deployment-assistant/
│
├─ README.md                              ← this file
├─ AGENTS.md                              ← orchestrator contract (4-pillar flow)
│
├─ spec/                                  ← SPEC LAYER (source of truth, machine-readable)
│  ├─ mdc/
│  │  ├─ skill.md                         mental model
│  │  ├─ validation.yaml                  Pillars 1+2 (detect/infer/ask)
│  │  ├─ license-matrix.yaml              licence ↔ feature map
│  │  ├─ preflight.md                     read-only az / Graph commands
│  │  ├─ playbook.md                      Pillar 3 phased deployment
│  │  └─ verification.yaml                Pillar 4 L1/L2/L3 + diagnostics + rollback
│  └─ _template/                          copy this to add a new product
│
├─ docs/                                  ← human-readable docs
│  ├─ architecture.md                     two-layer model + folder map
│  ├─ architecture-diagram.md             mermaid diagrams (components, flow, verify)
│  └─ integration.md                      wiring spec into Copilot / Foundry / Studio
│
├─ .github/                               ← RUNTIME LAYER
│  ├─ copilot-instructions.md             router (product detection)
│  └─ instructions/<product>/             per-product instruction blobs
│
├─ agents/                                ← RUNTIME LAYER
│  ├─ orchestrator.md                     shared persona + rules (lighter mirror of AGENTS.md)
│  └─ <product>-deployment.md             per-product personas
│
├─ skills/<product>/SKILL.md              ← RUNTIME LAYER (skill registration)
│
└─ demo-scripts/                          ← demo recording walkthroughs
   └─ mdc-demo-walkthrough.md
```

## Adding a new product

1. **Spec**: `cp -r spec/_template spec/<product>` and fill in the six files.
2. **Register** in [`AGENTS.md`](AGENTS.md) `## Product registry`.
3. **Runtime mirror** (so Copilot sees it):
   - `skills/<product>/SKILL.md` — skill definition
   - `agents/<product>-deployment.md` — agent persona
   - `.github/instructions/<product>/` — instruction files
   - Update routing in [`.github/copilot-instructions.md`](.github/copilot-instructions.md)
4. Update this README's Supported Products table.

## Recording a demo

See [`demo-scripts/mdc-demo-walkthrough.md`](demo-scripts/mdc-demo-walkthrough.md) for the MDC demo talk track and step-by-step recording guide.
