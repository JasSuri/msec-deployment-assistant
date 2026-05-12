# Deployment Assistant — Agent Configuration

An AI-driven Cloud Solution Architect that helps customers **understand → validate → deploy** Microsoft cloud security and management products without fear.

This repo holds the **agent context** (skills, validations, license matrices, playbooks) that drive the assistant. The conversational layer (GHCP / Copilot Studio / Foundry agent / etc.) reads these files at runtime.

> **Demo scope (v1):** Microsoft Defender for Cloud (MDC), persona = IT Admin.
> **Roadmap:** Intune, Entra, Purview, Sentinel, Defender XDR — each plugs into the same structure under `products/`.

---

## Folder structure

```
deployment-assistant/
├─ README.md                  ← you are here
├─ INTEGRATION.md             ← how to wire these files into the AI (GHCP / Foundry / Copilot Studio)
├─ AGENTS.md                  ← top-level agent contract (orchestrator behaviour, 3-pillar flow)
│
├─ products/                  ← one folder per product. Add new products here.
│  └─ mdc/
│     ├─ skill.md             ← what MDC is, capabilities, mental model (Pillar 1 source of truth)
│     ├─ validation.yaml      ← machine-readable Pillar 1 + 2 questions, gates, decisions
│     ├─ license-matrix.yaml  ← licence ↔ feature mapping (drives "what can I use today?")
│     ├─ preflight.md         ← CLI / Graph / REST commands the agent runs to detect state
│     └─ playbook.md          ← Pillar 3 phased deployment (commands + safety rails)
│
└─ _template/                 ← copy this when adding a new product
   ├─ skill.md
   ├─ validation.yaml
   ├─ license-matrix.yaml
   ├─ preflight.md
   └─ playbook.md
```

## How the assistant uses it (3-pillar flow)

```
User: "I want to use MDC to protect my cloud — help me plan and deploy."
        │
        ▼
   AGENTS.md  ─── routes to product = mdc
        │
        ▼
┌─ PILLAR 1: UNDERSTAND ─────────────────────────────┐
│  Source: products/mdc/skill.md                     │
│  Driver: products/mdc/validation.yaml (pillar_1)   │
│  Goal:   correct misconceptions, confirm fit       │
└────────────────────────────────────────────────────┘
        │
        ▼
┌─ PILLAR 2: VALIDATE ───────────────────────────────┐
│  Driver: products/mdc/validation.yaml (pillar_2)   │
│  Detect: products/mdc/preflight.md (live CLI/Graph)│
│  Map:    products/mdc/license-matrix.yaml          │
│  Goal:   confirm licences + roles + env are ready  │
└────────────────────────────────────────────────────┘
        │
        ▼
┌─ PILLAR 3: GUIDE ──────────────────────────────────┐
│  Driver: products/mdc/playbook.md                  │
│  Goal:   execute phased deployment with safety     │
└────────────────────────────────────────────────────┘
```

See [INTEGRATION.md](INTEGRATION.md) for exact wiring instructions for the AI runtime.

## Adding a new product (e.g. Intune)

1. `cp -r _template products/intune`
2. Fill in `skill.md` (what Intune is, capabilities, common misconceptions).
3. Fill in `validation.yaml` (Pillar 1 + 2 questions specific to Intune).
4. Fill in `license-matrix.yaml` (M365 E3 / E5 / EMS / Intune Plan 1+2 / Suite).
5. Fill in `preflight.md` (Graph calls: `/deviceManagement`, `/policies/...`).
6. Fill in `playbook.md` (enrolment → compliance → CA → app protection).
7. Register the product in `AGENTS.md` under `## Product registry`.

The agent picks it up automatically — no orchestrator code changes.
