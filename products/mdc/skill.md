# Skill — Microsoft Defender for Cloud (MDC)

> Authoritative mental model the agent must hold about MDC. All Pillar 1 answers come from here.
> Sources: [What is MDC](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-cloud-introduction) · [Defender plans](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-cloud-introduction#protect-cloud-workloads) · [Pricing](https://azure.microsoft.com/pricing/details/defender-for-cloud/)

## What MDC is (one sentence)
A **Cloud-Native Application Protection Platform (CNAPP)** that combines posture management (CSPM), workload protection (CWPP), multi-cloud coverage, and DevOps security across Azure, AWS, GCP and hybrid via Azure Arc.

## What MDC is **not**
| It is NOT | The right product is |
|---|---|
| An endpoint AV for laptops/desktops | Microsoft Defender for Endpoint (MDE) |
| An email / phishing filter | Microsoft Defender for Office 365 |
| A SIEM/SOAR | Microsoft Sentinel (MDC *feeds* Sentinel) |
| A DLP / data classification tool | Microsoft Purview |
| Identity / MFA / Conditional Access | Microsoft Entra ID |
| A network firewall manager (it recommends, doesn't auto-apply) | Azure Firewall + Azure Policy |

## Capability map (what the agent can offer)

### Tier 1 — Foundational CSPM (free, on by default)
- Secure Score
- Security recommendations across Azure resources
- Asset inventory
- Regulatory compliance dashboard (MCSB built-in)

### Tier 2 — Defender CSPM (paid, per billable resource)
- Attack path analysis
- Cloud security graph
- Agentless vulnerability scanning (VMs, containers)
- Agentless secret scanning
- Permissions Management (CIEM) integration
- Data-aware security posture
- Governance & ServiceNow integration

### Tier 3 — Defender workload plans (paid, per resource type)
| Plan | Protects |
|---|---|
| Defender for Servers P1 | MDE for Servers (anti-malware/EDR), file integrity monitoring lite |
| Defender for Servers P2 | All P1 + MDVM premium, JIT VM access, FIM, free 500 MB Log Analytics ingest |
| Defender for SQL | Azure SQL DB, Managed Instance, SQL on VMs |
| Defender for Storage | Activity monitoring + optional Malware Scanning + Sensitive Data Discovery |
| Defender for Containers | AKS / EKS / GKE / Arc-enabled K8s — runtime protection, image scan, K8s posture |
| Defender for App Service | App Service plan threat detection |
| Defender for Key Vault | Anomalous access detection |
| Defender for Resource Manager | Suspicious ARM operations |
| Defender for DNS | Malicious DNS lookups (deprecated standalone — folded into Servers) |
| Defender for APIs | API Management runtime protection |
| Defender for AI Services | Azure OpenAI / AI Services threat detection |
| Defender for Cosmos DB | Cosmos DB threat detection |

### Tier 4 — Multi-cloud & hybrid
- AWS connector (CloudFormation) → CSPM + CWPP for AWS accounts.
- GCP connector (Terraform / `gcloud`) → CSPM + CWPP for GCP projects.
- Azure Arc → on-prem servers / non-Azure VMs into Defender for Servers.

### Tier 5 — DevOps security
- GitHub & Azure DevOps connectors → IaC scanning, secret scanning, code-to-cloud mapping.

## What changes when MDC is enabled (fear reduction)
| Change | Severity | Reversible? |
|---|---|---|
| Foundational CSPM on subscription | None | ✅ |
| Defender plan toggled on | Low–Medium (cost + agent install) | ✅ toggle off |
| Auto-provisioned agents (AMA, MDE for Servers, MDVM, Defender sensor) | Low (<2% CPU) | ✅ uninstall |
| Azure Policy security initiative assigned | Low (audit-only by default) | ✅ remove assignment |
| Continuous export to Sentinel / Event Hub | Low (downstream cost) | ✅ |
| AWS/GCP connector creates **read-only** IAM role / service account | Medium | ✅ delete connector |

## What does NOT change
Workloads, application behaviour, network configuration, end-user experience, existing Azure Policy assignments, data residency (your chosen LA workspace region).

## Common IT-admin misconceptions (Pillar 1 red flags)
1. ❌ "MDC = MDE" → Different products. MDC integrates MDE *for servers only*.
2. ❌ "MDC replaces Sentinel" → No. MDC is a data source for Sentinel.
3. ❌ "MDC will fix my NSGs automatically" → It recommends; it does not auto-apply network changes.
4. ❌ "Free tier covers everything" → Foundational CSPM is free; Defender CSPM and workload plans are paid.
5. ❌ "It only covers Azure" → AWS, GCP, on-prem (Arc) all supported.
6. ❌ "Enabling it will break my workloads" → Agents are passive; no runtime impact by default.

## Required Microsoft Learn references (the agent must cite from this list)
- https://learn.microsoft.com/azure/defender-for-cloud/defender-for-cloud-introduction
- https://learn.microsoft.com/azure/defender-for-cloud/permissions
- https://learn.microsoft.com/azure/defender-for-cloud/concept-defender-for-cloud-permissions
- https://learn.microsoft.com/azure/defender-for-cloud/enable-defender-for-cloud
- https://learn.microsoft.com/azure/defender-for-cloud/quickstart-onboard-aws
- https://learn.microsoft.com/azure/defender-for-cloud/quickstart-onboard-gcp
- https://learn.microsoft.com/azure/defender-for-cloud/concept-cloud-security-posture-management
- https://azure.microsoft.com/pricing/details/defender-for-cloud/
