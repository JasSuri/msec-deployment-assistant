# MDC Deployment Playbook — Pillar 3 (GUIDE)

> Only enter this playbook after both `validation.yaml` gates are GREEN. Every state-changing command must be confirmed by the user before execution. Every phase has a rollback.

Convention: `{{subId}}`, `{{mgId}}`, `{{workspaceId}}`, `{{rgName}}` are placeholders the agent fills from preflight.

---

## Phase 0 — Baseline (free, no risk)
**phase_id:** `phase_0_baseline`   **verify:** [verification.yaml#phase_0_baseline](verification.yaml)
**Goal:** Foundational CSPM enabled tenant-wide. Secure Score baseline captured.

**Required role:** Security Admin or Owner at MG / sub scope.

**Apply:**
```bash
# Ensure provider is registered
az provider register --namespace Microsoft.Security

# Foundational CSPM is on by default; explicitly confirm + enable Defender CSPM if desired
az security pricing show -n CloudPosture --subscription {{subId}}
```

**Verify:** Run `verification.yaml#phase_0_baseline` → L1 (provider Registering/Registered) → L2 (Registered) → L3 deferred (assessments populate).

**Rollback:** N/A (free tier; cannot be disabled without disabling MDC entirely).

---

## Phase 1 — Pilot (one non-prod sub, paid plans, 30-day trial)
**phase_ids:** `phase_1_pilot_servers_p2`, `phase_1_pilot_storage_v2`, `phase_1_workspace_setting`   **verify:** [verification.yaml](verification.yaml)
**Goal:** Validate alerts, agent behaviour, real cost on ONE subscription.

**Apply:**
```bash
# Defender for Servers P2 (uses 30-day trial)
az security pricing create -n VirtualMachines --tier Standard --subplan P2 --subscription {{pilotSubId}}

# Defender for Storage V2
az security pricing create -n StorageAccounts --tier Standard --subplan DefenderForStorageV2 --subscription {{pilotSubId}}

# Set workspace for Defender for Servers data
az security workspace-setting create -n default --target-workspace {{workspaceResourceId}} --subscription {{pilotSubId}}
```

**Verify:** Run the three matching `verification.yaml` blocks. L1+L2 must be ✅ before declaring Phase 1 complete. L3 (sample VM gets MDE extension, workspace receives SecurityEvent rows) is deferred — surface as `pending` and schedule follow-up at the listed `lag`.

**Rollback:** Each phase id has its own `rollback` block in `verification.yaml` with a `verify_rollback` check. Always confirm rollback verified before reporting.

**Agent guardrail:** before applying, print: *"This will start a 30-day free trial on subscription {{pilotSubId}}. After 30 days you will be billed at ~$X/server/month for ~N servers. Continue? (yes/no)"*

---

## Phase 2 — Tenant-wide CSPM
**phase_id:** `phase_2_tenant_cspm`   **verify:** [verification.yaml#phase_2_tenant_cspm](verification.yaml)
**Goal:** Defender CSPM (paid) across the whole MG → attack paths, agentless scan, secret scan.

**Apply at MG scope via Azure Policy (preferred over per-sub loops):**
```bash
# Built-in initiative: "Configure Microsoft Defender CSPM plan"
az policy assignment create \
  --name "enable-defender-cspm" \
  --policy-set-definition "{{builtinInitiativeId}}" \
  --scope /providers/Microsoft.Management/managementGroups/{{mgId}} \
  --mi-system-assigned --location eastus
```

**Verify:** Run `verification.yaml#phase_2_tenant_cspm`. L2 polls a spot subscription for up to 30 min (policy compliance scan cadence). L3 (attack-path findings) deferred 24h.

**Rollback:** delete the policy assignment + flip plans back to Free — see `verification.yaml#phase_2_tenant_cspm.rollback`.

---

## Phase 3 — Workload protection expansion
Roll forward additional plans **one resource type at a time**, one sub at a time first:
- Containers (requires AKS Defender sensor — auto-provisioned)
- Key Vault, App Service, Resource Manager, APIs, AI Services, Cosmos DB

For each:
1. Enable on pilot sub → wait 24h → review alert volume.
2. Promote to MG via policy if noise is acceptable.

---

## Phase 4 — Multi-cloud
**phase_id:** `phase_4_aws_connector` (mirror for GCP)   **verify:** [verification.yaml#phase_4_aws_connector](verification.yaml)
**AWS:**
```bash
# Use the portal-generated CloudFormation template — DO NOT hand-craft IAM
# 1. In Azure portal: MDC → Environment settings → Add environment → AWS
# 2. Download the generated CloudFormation template
# 3. AWS admin runs it in management account (or each member account)
```
Verify: `az security security-connector list --subscription {{subId}} -o table` → AWS connector status `Healthy`.

**GCP:** identical pattern with Terraform / `gcloud` script.

**Rollback:** delete the connector + remove the IAM role / service account in AWS/GCP.

---

## Phase 5 — Hybrid (Azure Arc)
**phase_id:** `phase_5_arc_onboarding`   **verify:** [verification.yaml#phase_5_arc_onboarding](verification.yaml)
Onboard servers, then Defender for Servers covers them automatically.

```bash
# Per server (interactive):
azcmagent connect --resource-group {{rgName}} --tenant-id {{tenantId}} --location {{region}} --subscription-id {{subId}}

# At scale: use the Service Principal-based onboarding script generated by Arc portal.
```

**Rollback:** `azcmagent disconnect` then delete the Connected Machine resource.

---

## Phase 6 — SOC integration
**phase_id:** `phase_6_workflow_automation`   **verify:** [verification.yaml#phase_6_workflow_automation](verification.yaml)
- Continuous Export → Sentinel / Event Hub.
- Workflow Automation (Logic Apps) for high-severity alerts.
- ServiceNow ITSM (Defender CSPM Governance).

```bash
az security automation create --name <name> --resource-group <rg> --location <loc> \
  --scopes "[{description:'sub',scopePath:/subscriptions/{{subId}}}]" \
  --sources @sources.json --actions @actions.json
```

---

## Phase 7 — Compliance & ongoing
**phase_id:** `phase_7_compliance_assignment`   **verify:** [verification.yaml#phase_7_compliance_assignment](verification.yaml)
- Assign regulatory standards (CIS, NIST 800-53, PCI-DSS, ISO 27001, SOC2).
- Subscribe to monthly Secure Score review.
- Tune suppression rules.

```bash
az policy assignment create --name cis-azure-2.0 --policy-set-definition {{cisInitiativeId}} \
  --scope /providers/Microsoft.Management/managementGroups/{{mgId}}
```

---

## Universal safety rules the agent MUST follow
1. **Pilot first, scale second.** Never enable a paid plan tenant-wide before 24h on a pilot sub.
2. **Print → confirm → execute.** Echo every command and its blast radius before running.
3. **Always offer rollback.** Show the rollback command at the same time as the apply command.
4. **Cost ceiling.** Stop and ask if estimated monthly cost crosses the user's stated budget.
5. **Idempotency.** Re-running any command in this playbook must not double-charge or break state.
