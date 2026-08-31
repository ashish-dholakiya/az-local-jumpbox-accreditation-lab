# Azure Local Accreditation Lab Dependency Register

## Purpose

This document captures the dependencies identified during the Azure Local accreditation lab implementation. It supports technical execution, implementation evidence, and presentation preparation.

The register explains what was required, why it mattered, the current validation state, and how the dependency story should be positioned during the accreditation presentation.

The governing accreditation scope remains `docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md`. This dependency register is a supporting implementation control document.

---

## 1. Presentation Positioning

Recommended presentation section:

**Implementation Readiness and Dependency Controls**

Recommended message:

> Before deploying the Azure Local Jumpstart LocalBox environment, we validated subscription access, region and quota readiness, tooling, Azure resource-provider registrations, source reproducibility, and execution continuity. Dependencies that were not obvious at the beginning of the implementation were converted into a formal dependency register so that the proof of concept remains repeatable, auditable, and controlled.

---

## 2. Dependency Register

| Category | Dependency | Why it matters | Status |
| --- | --- | --- | --- |
| Subscription | Correct company accreditation subscription selected | Prevents deployment into an incorrect or personal test subscription. | PASS |
| Access | Effective Owner RBAC | Required for provider registration, resource group operations, and LocalBox deployment activities. | PASS |
| Region | Central India selected | Keeps the lab aligned to the selected deployment region. | PASS |
| Quota | Total Regional vCPU quota | LocalBox requires a large Azure VM, so sufficient regional quota is mandatory. | PASS |
| Quota | Standard Esv6 Family vCPU quota | Required for `Standard_E32s_v6`. | PASS |
| SKU | `Standard_E32s_v6` availability | Required by the verified LocalBox deployment template. | PASS |
| Tooling | Git | Required to clone and pin the official Microsoft Jumpstart source. | PASS |
| Tooling | PowerShell 7 | Used for controlled execution from VS Code. | PASS |
| Tooling | Azure CLI | Required for authentication, subscription validation, provider operations, resource verification, and deployment. | PASS |
| Tooling | Bicep CLI | Required for the official LocalBox Bicep deployment workflow. | PASS |
| Source Control | Accreditation repository cloned locally | Provides persistent access to the accreditation source of truth and implementation documentation. | PASS |
| Source Control | Microsoft `azure_arc` repository cloned separately | Keeps Microsoft deployment source separate from accreditation documentation and avoids nested Git repository issues. | PASS |
| Source Control | Microsoft repository pinned to exact commit | Avoids dependence on a moving source branch and improves repeatability. | PASS |
| Runtime Source | Runtime `githubBranch` behavior | The LocalBox template uses this value for raw GitHub runtime artifacts. The exact verified commit must be used instead of a moving `main` reference. | PASS |
| Provider | `Microsoft.AzureStackHCI` | Required for Azure Local resource registration and management. | PASS |
| Provider | `Microsoft.HybridCompute` | Supports Azure Arc-enabled resource integration. | PASS |
| Provider | `Microsoft.HybridConnectivity` | Supports hybrid connectivity functions used by Azure Arc scenarios. | PASS |
| Provider | `Microsoft.ResourceConnector` | Supports Arc Resource Bridge and related Azure Local integration. | PASS |
| Provider | `Microsoft.Kubernetes` | Required by supporting Arc/Kubernetes resource integration where referenced. | PASS |
| Provider | `Microsoft.KubernetesConfiguration` | Required for Kubernetes configuration integration where referenced. | PASS |
| Provider | `Microsoft.ExtendedLocation` | Required for extended/custom location concepts used by Azure Local. | PASS |
| Provider | `Microsoft.Attestation` | Supports Azure Local attestation-related integration. | PASS |
| Provider | `Microsoft.Compute` | Required for VM deployment, sizing, and quota validation. | PASS |
| Provider | `Microsoft.Network` | Required for VNet, NIC, public IP, NAT gateway, and related network resources. Initially discovered as `NotRegistered` during direct Bicep dependency validation. | PASS |
| Provider | `Microsoft.OperationalInsights` | Required for Log Analytics workspace and monitoring resources. Initially discovered as `NotRegistered` during direct Bicep dependency validation. | PASS |
| Provider | `Microsoft.Storage` | Required for staging/storage resources used by the LocalBox deployment. | PASS |
| Provider | `Microsoft.KeyVault` | Supporting provider registered during subscription readiness validation. | PASS |
| Provider | `Microsoft.Insights` | Supporting provider registered for Azure Monitor-related integration. | PASS |
| Execution Continuity | Dedicated Azure CLI profile | Isolates the accreditation Azure login/subscription context from other Azure projects on the same workstation. | PASS |
| Execution Continuity | Local VS Code workflow | Avoids dependency on ephemeral Cloud Shell sessions for long-running accreditation work. | PASS |
| Execution Continuity | Resume helper script | Allows the implementation session to be safely resumed after terminal closure or laptop restart. | PASS |
| Security | Secure Windows administrator password input | Mandatory LocalBox deployment parameter. The value must not be stored in the public repository, chat, or evidence screenshots. | PENDING |
| Deployment | Final LocalBox deployment command validation | Required before billable LocalBox resources are created. | PENDING |
| Deployment | LocalBox resource deployment | Creates the Azure Local Jumpstart LocalBox lab resources and begins billable workload deployment. | NOT STARTED |

---

## 3. Dependencies Identified During Implementation

Some dependencies were known before implementation, while others became visible only after inspecting the actual Microsoft LocalBox source and running readiness checks.

### Initially known prerequisites

- Correct Azure subscription.
- Owner access.
- Target region.
- VM SKU and quota.
- Azure Local and Azure Arc provider registrations.
- Azure CLI and Bicep tooling.

### Dependencies discovered during deeper validation

#### `Microsoft.Network`

The direct LocalBox Bicep dependency scan showed that `Microsoft.Network` is referenced by the official deployment modules.

Initial state:

```text
Microsoft.Network : NotRegistered
```

Final state:

```text
Microsoft.Network : Registered
```

#### `Microsoft.OperationalInsights`

The direct LocalBox Bicep dependency scan showed that `Microsoft.OperationalInsights` is required for Log Analytics resources.

Initial state:

```text
Microsoft.OperationalInsights : NotRegistered
```

Final state:

```text
Microsoft.OperationalInsights : Registered
```

#### Runtime GitHub source reference

The LocalBox template defaults `githubBranch` to `main` and builds raw GitHub artifact URLs from that value.

A runtime artifact was tested against the exact verified Microsoft source commit:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

Verified result:

```text
Pinned runtime artifact reachable: True
HTTP Status: 200
```

This allows the deployment to use the exact commit reference for runtime artifacts instead of relying on a moving `main` branch.

---

## 4. Execution Continuity Dependency

Cloud Shell was initially used for readiness and source validation. During implementation it became clear that an ephemeral shell session is not ideal for work performed in multiple short sessions.

The execution model was therefore moved to a persistent VS Code workflow.

Verified local components:

- Git `2.55.0`.
- PowerShell `7.6.5`.
- Azure CLI `2.89.1`.
- Bicep CLI `0.46.1`.
- Local accreditation repository.
- Separate local Microsoft `azure_arc` repository.
- Dedicated Azure CLI profile for accreditation.
- Resume helper script for state validation.

This change did not alter the accreditation architecture or deployment method. It only improved execution continuity and reduced session-related risk.

---

## 5. Security Dependencies and Controls

The following values must not be committed to the public accreditation repository:

- Full Azure subscription ID.
- Tenant ID.
- Azure Local resource-provider object ID.
- Windows administrator password.
- Credentials, tokens, or secrets.
- Confidential internal documentation.
- Sensitive screenshots.

The Windows administrator password required by the LocalBox template remains a runtime-only secure input.

---

## 6. Current Dependency Readiness

### Completed

- Subscription selection and isolation.
- Owner access validation.
- Central India region decision.
- VM SKU availability.
- Regional and VM-family quota.
- Core Azure Local and Azure Arc providers.
- Direct LocalBox Bicep provider dependencies.
- Azure CLI.
- Bicep CLI.
- Official Microsoft source pinning.
- Runtime artifact exact-commit reachability.
- Persistent VS Code execution environment.
- Resume workflow.

### Pending before billable deployment

1. Secure Windows administrator password input.
2. Final LocalBox deployment command validation.

### Not started

- Billable LocalBox deployment.
- Azure Local cluster deployment/review validation.
- Azure Arc validation.
- Azure Local VM creation and lifecycle validation.
- Logical workload networking validation.
- Azure monitoring and management validation.
- Azure Local update and lifecycle validation.

---

## 7. Key Lessons Learned

1. Provider dependencies should be validated against the actual deployment source rather than assumed only from an initial prerequisite list.
2. Local repository pinning alone does not guarantee runtime reproducibility when deployment scripts download artifacts from GitHub. Runtime references must also be validated.
3. Cloud Shell is useful for quick checks, but a persistent local VS Code workflow is more suitable for an accreditation lab completed over multiple working sessions.
4. Azure CLI context should be isolated when multiple Azure projects and accounts exist on the same workstation.
5. Secrets must remain outside the public repository and presentation evidence.
6. Turning hidden prerequisites into a dependency register makes implementation delays explainable as controlled risk reduction rather than unstructured troubleshooting.

---

## 8. Presentation Content

This document must feed the accreditation presentation.

### Suggested slide: Implementation Readiness Controls

- Validated company subscription, Owner access, Central India region, quota, and VM SKU availability.
- Validated and registered required Azure Local, Arc, Compute, Network, Storage, Monitoring, and Log Analytics resource providers.
- Moved execution from Cloud Shell to a persistent VS Code workflow to avoid session loss.
- Isolated the accreditation Azure CLI profile from other Azure projects on the workstation.
- Pinned the official Microsoft LocalBox source and validated runtime artifact reachability.

### Suggested slide: Dependency Register and Risk Reduction

- Converted implementation prerequisites into a formal dependency register.
- Identified `Microsoft.Network` and `Microsoft.OperationalInsights` before billable deployment.
- Prevented source drift by validating both local source pinning and runtime GitHub artifact references.
- Kept secrets and sensitive identifiers outside the public evidence repository.

### Suggested slide: Current Build Readiness

- Subscription, access, region, quota, tooling, source, runtime references, and direct provider dependencies: **Ready**.
- Pending before deployment: secure password input and final deployment command validation.
- LocalBox workload deployment: **Not Started**.

---

## 9. Governance

This dependency register supports Accreditation Activity 3: Azure Local Proof of Concept using Microsoft Azure Arc Jumpstart LocalBox.

It does not replace the accreditation source-of-truth document. All implementation and presentation decisions must remain traceable to the required accreditation activities and customer scenario.