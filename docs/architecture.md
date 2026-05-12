# Deployment Assistant — Architecture

An AI-driven Cloud Solution Architect that helps customers **understand → validate → deploy → verify** Microsoft cloud security products without fear.

> **Demo scope (v1):** Microsoft Defender for Cloud (MDC), persona = IT Admin.
> **Roadmap:** Intune, Entra, Purview, Defender XDR, Sentinel — each plugs into the same `spec/` pattern.

---

## The two-layer model

This repo holds the assistant in **two intentional layers**:

| Layer | Purpose | Files |
|---|---|---|
| **Spec** (machine-readable, source of truth) | Deterministic contract the AI consumes — questions, gates, detect commands, licence maps, verify steps, rollback. Versioned and reviewed like code. | [`AGENTS.md`](../AGENTS.md), [`spec/<product>/*`](../spec) |
| **Runtime** (Copilot-loadable) | Files Copilot / Foundry / Copilot Studio actually loads at conversation time — agent personas, instruction blobs, skill registrations. | [`agents/`](../agents), [`skills/`](../skills), [`.github/copilot-instructions.md`](../.github/copilot-instructions.md), [`.github/instructions/`](../.github/instructions) |

**Why two layers?**
- The spec is the single source of truth — it's where licence facts, role matrices, verify commands, and gates live. One change here propagates everywhere.
- The runtime is what Copilot loads in a chat. Different runtimes (GHCP, Foundry, Copilot Studio) want different file shapes. The runtime layer is the adapter.
- Currently the runtime layer is hand-authored (Jas's structure). The spec layer is the new contract that backs it. See [integration.md](integration.md) for how to wire one to the other.

---

## The 4-pillar flow (what the agent does)

```
User prompt
    │
    ▼
PILLAR 1 — UNDERSTAND       overcome fear via clarity
    │   1 opener question, red-flag/redirect matrix
    │   sources: spec/<p>/skill.md, spec/<p>/validation.yaml#pillar_1
    ▼
PILLAR 2 — VALIDATE         no surprises during deployment
    │   detect → infer → ask (in that order, never the other way)
    │   sources: spec/<p>/validation.yaml#pillar_2, spec/<p>/preflight.md, spec/<p>/license-matrix.yaml
    ▼
PILLAR 3 — GUIDE            phased, reversible deployment
    │   print → confirm → execute, smallest blast radius first
    │   sources: spec/<p>/playbook.md
    ▼
PILLAR 4 — VERIFY           prove the change actually took
    │   L1 acknowledge (sync) → L2 state (poll) → L3 functional (deferred)
    │   on fail: diagnostics → rollback → verify_rollback
    │   sources: spec/<p>/verification.yaml
    ▼
Phase complete  ──→  next phase OR session done
```

Full contract for each pillar lives in [`AGENTS.md`](../AGENTS.md).

---

## Folder layout

```
deployment-assistant/
│
├─ README.md                              ← repo entry point (Jas's overview)
├─ AGENTS.md                              ← orchestrator contract (4-pillar flow, hard rules)
├─ .gitignore
│
├─ spec/                                  ← SPEC LAYER (source of truth)
│  │
│  ├─ <product>/                          ← one folder per product
│  │  ├─ skill.md                         ← mental model, "what it is / is not", Pillar 1 source
│  │  ├─ validation.yaml                  ← Pillar 1 + Pillar 2 (detect / infer / ask)
│  │  ├─ license-matrix.yaml              ← licence ↔ feature map
│  │  ├─ preflight.md                     ← read-only az CLI + Graph commands
│  │  ├─ playbook.md                      ← Pillar 3 phased deployment
│  │  └─ verification.yaml                ← Pillar 4 L1/L2/L3 verify, diagnostics, rollback
│  │
│  ├─ mdc/                                ← ✅ v1 demo (complete)
│  └─ _template/                          ← copy this folder to add a new product
│
├─ docs/                                  ← humans read these
│  ├─ architecture.md                     ← this file
│  └─ integration.md                      ← how to wire spec/ into Copilot / Foundry / Studio
│
├─ .github/                               ← RUNTIME LAYER (Copilot loads these)
│  ├─ copilot-instructions.md             ← router (product detection)
│  └─ instructions/<product>/*.md         ← per-product instruction blobs
│
├─ agents/                                ← RUNTIME LAYER
│  ├─ orchestrator.md                     ← shared persona + rules (lighter mirror of AGENTS.md)
│  └─ <product>-deployment.md             ← per-product agent persona
│
├─ skills/<product>/SKILL.md              ← RUNTIME LAYER (skill registration)
│
└─ demo-scripts/                          ← demo recording walkthroughs
```

## Adding a new product (e.g. Intune)

1. `cp -r spec/_template spec/intune`
2. Fill in the six spec files (skill, validation, license-matrix, preflight, playbook, verification).
3. Register the product in [`AGENTS.md`](../AGENTS.md) under `## Product registry`.
4. (Optional) Mirror to runtime layer for Copilot: create `agents/intune-deployment.md`, `skills/intune/SKILL.md`, `.github/instructions/intune/*.instructions.md` from the spec.
5. Update the routing table in [`.github/copilot-instructions.md`](../.github/copilot-instructions.md).

---

## See also
- [`AGENTS.md`](../AGENTS.md) — the orchestrator contract (4 pillars, modes, gates)
- [`docs/integration.md`](integration.md) — exact wiring for GHCP / Foundry / Copilot Studio
- [`spec/mdc/`](../spec/mdc) — the v1 demo product, fully filled in
- [`spec/_template/`](../spec/_template) — copy this when adding a new product
