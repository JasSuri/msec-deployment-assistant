# Integration Guide — wiring this repo into your AI runtime

This repo holds **agent context** as plain files. The conversational layer (whatever you choose) reads them at runtime. Below are the three most likely runtimes for the demo.

## What the AI needs from this repo, in order

1. **System prompt** ← `AGENTS.md` (the orchestrator contract).
2. **Loaded skills** ← `spec/mdc/skill.md` (and any other product folders you register).
3. **Structured validation driver** ← `spec/mdc/validation.yaml` (the question/check engine).
4. **Tool-callable references** ← `spec/mdc/preflight.md`, `playbook.md`, `license-matrix.yaml` (treat as retrievable docs OR pre-load into context if the model can hold it).

> Rule of thumb: `skill.md` + `validation.yaml` + `license-matrix.yaml` should always be in context. `preflight.md` and `playbook.md` can be retrieved on demand.

---

## Option 0 — GitHub Copilot CLI plugin

This repository can now be installed directly as a GitHub Copilot CLI plugin because it ships with:

- `plugin.json` at the repository root
- plugin-compatible agent profiles in `agents/*.agent.md`
- plugin-compatible skills in `skills/*/SKILL.md`
- `.github/plugin/marketplace.json` for marketplace registration

### Direct install

```shell
copilot plugin install OWNER/REPO
```

Or from a Git URL:

```shell
copilot plugin install https://github.com/OWNER/REPO.git
```

### Marketplace registration

Because the repo includes `.github/plugin/marketplace.json`, it can also be registered as a plugin marketplace and then browsed or installed from there.

### Content model inside the plugin

- `spec/` remains the canonical source for structured product packs.
- `agents/` and `skills/` are the Copilot CLI plugin surface.
- `.github/copilot-instructions.md` keeps repository-level routing deterministic when the repo is used directly without plugin installation.

---

## Option A — GitHub Copilot Chat (skills.md / agents.md pattern, your colleague's setup)

Map this repo directly:

| Your colleague's file | Source from this repo |
|---|---|
| `agents.md` (top level) | `deployment-assistant/AGENTS.md` |
| `skills/mdc.skill.md` | `deployment-assistant/spec/mdc/skill.md` + the YAML files appended as fenced code blocks |

Concrete steps:
1. Copy `AGENTS.md` to the root of the repo your colleague is using as `.github/copilot-instructions.md` (or `agents.md`, depending on convention).
2. For each product folder, create a single composite skill file the model can see in one read:
   ```
   skills/mdc.skill.md   ← concat of: skill.md + ```yaml validation.yaml``` + ```yaml license-matrix.yaml``` + preflight.md + playbook.md
   ```
   *(Or leave them split if your runtime supports loading multiple skill files per turn — preferred.)*
3. Tell Copilot to read these files first when the user mentions MDC / Defender for Cloud.

## Option B — Azure AI Foundry / Copilot Studio agent

1. **Instructions** field of the agent ← contents of `AGENTS.md`.
2. **Knowledge** (file upload / RAG index) ← upload the entire `deployment-assistant/` folder. Foundry will chunk + retrieve.
3. **Tools / Actions:** register two function tools the model can call:
   - `run_preflight(subscription_id)` → executes the read-only commands from `preflight.md` against the customer's subscription via Azure CLI / SDK and returns structured JSON.
   - `apply_phase(phase_id, subscription_id, confirm=true)` → executes the corresponding block from `playbook.md`. Must require `confirm=true` from the user.
4. **Citations:** turn on file citation so every claim links back to the file in this repo.

## Option C — Local dev with GHCP custom agent (`.agent.md`)

```
.github/
└── agents/
    └── deployment-assistant.agent.md   ← AGENTS.md content + frontmatter pointing to product files
```

Use the agent customisation skill (already loaded for me) if you want me to scaffold the `.agent.md` frontmatter — say the word.

---

## How the validation drives the conversation (concrete example)

Given the demo prompt: *"I want to use MDC to protect my cloud environments, help me plan and deploy"*:

| Turn | Agent action | File it reads | Output to user |
|---|---|---|---|
| 1 | Loads MDC skill, picks first p1 question | `skill.md`, `validation.yaml#pillar_1.questions[0]` | "In your own words, what do you think MDC does?" |
| 2 | User: *"It's antivirus for VMs."* → red_flag matches `antivirus` | `validation.yaml#p1.1.red_flags` | Delivers `corrective_message`, re-asks. |
| 3 | Pillar 1 cleared → enters Pillar 2. Calls `run_preflight` tool. | `preflight.md` | Returns `PREFLIGHT REPORT` block. |
| 4 | Cross-references detected plans against `license-matrix.yaml`. | `license-matrix.yaml` | "You already pay for Defender for Servers P2 but it's only on 1 of 4 subs — free upgrade in posture by enabling it on the other 3." |
| 5 | Detects user has only Contributor. Triggers `b.no_security_admin`. | `validation.yaml#blockers` | "You'll need Security Admin via PIM. Want me to draft the request? Meanwhile we can start free CSPM." |
| 6 | Both gates green → enters Pillar 3 Phase 0. | `playbook.md#phase-0` | Prints command, asks confirm, executes via `apply_phase` tool. |

---

## Demo Azure environment prep checklist

To make the colleague's demo work end-to-end against the [MDC lab module](https://github.com/Azure/Microsoft-Defender-for-Cloud/blob/main/Labs/Modules/Module-1-Preparing-the-Environment.md#exercise-2-provisioning-resources):

- [ ] One Azure subscription dedicated to the demo (non-prod).
- [ ] Provision the lab resources from Module 1 Exercise 2 (VMs, SQL, Storage, AKS, Key Vault) — these become the targets MDC protects.
- [ ] A second subscription left **empty** to demo "tenant-wide rollout via MG".
- [ ] One management group containing both subs.
- [ ] Demo user identity with **Contributor** at sub scope (deliberately under-privileged → triggers `p2.7` role gap → great demo moment for the assistant to recommend PIM).
- [ ] PIM-eligible **Security Admin** assignment ready to activate (so the assistant can show the role-elevation path live).
- [ ] A Log Analytics workspace pre-created (or let MDC auto-create — either path is demo-able).
- [ ] Defender plans set to **Free** at the start so the assistant has something to enable.
- [ ] Optional: one AWS sandbox account + one GCP project to demo multi-cloud Phase 4.
- [ ] Optional: 1–2 Arc-onboarded VMs for hybrid demo Phase 5.
- [ ] Azure CLI ≥ 2.60 + `az extension add -n security` installed in the demo shell.
- [ ] Copilot / agent runtime configured per Option A/B/C above with this folder loaded.

---

## How to use the existing human-readable validation doc

`DeploymentAssistant-Validation.md` (the one you already have) is the **narrative** version — perfect for stakeholder reviews and CSA training. `validation.yaml` is the **executable** version that the AI consumes. Keep both: change the YAML when behaviour changes; regenerate the markdown summary from the YAML if you want them perfectly in sync (a small script can do this — ask if you want it).
