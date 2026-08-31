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

## 2. Dependency Classification

To avoid confusing Microsoft LocalBox implementation details with prerequisites required in this accreditation lab, dependencies are classified into three groups.

### 2.1 Our Deployment Dependencies

These are prerequisites that must exist or be validated in the accreditation environment before the LocalBox deployment can proceed safely.

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
| Tooling | `Az.Resources` PowerShell module `10.1.0` | Provides `New-AzResourceGroupDeployment`, matching the Azure PowerShell deployment pattern used by Microsoft's LocalBox validation pipeline. | PASS |
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

### 2.2 Microsoft LocalBox Solution Components

These components are part of, or directly referenced by, the Microsoft LocalBox implementation. They should not automatically be described as separate accreditation prerequisites unless they require an explicit dependency action in our environment.

| Microsoft LocalBox component | Role in the solution | How we use or treat it |
| --- | --- | --- |
| Microsoft `azure_arc` GitHub repository | Official source for Azure Arc Jumpstart LocalBox | Used as the authoritative Microsoft deployment source and pinned to a verified commit. |
| Bicep templates | Define Azure resources for LocalBox | Used as the official deployment mechanism. |
| PowerShell automation | Performs bootstrap, configuration, and supporting automation | Treated as Microsoft solution automation rather than separately reinvented scripts. |
| Runtime GitHub artifacts | Scripts and supporting artifacts are retrieved through raw GitHub URLs | Runtime reference was validated against the exact pinned commit. |
| Azure Compute | Provides the large Azure VM used to host the LocalBox environment | Subscription quota and supported VM SKU were validated before deployment. |
| Azure Networking | Provides VNet, NIC, public IP, NAT, and related networking | `Microsoft.Network` was identified and registered before deployment. |
| Azure Storage | Provides staging/storage resources used during deployment | Direct Bicep provider dependency validated. |
| Log Analytics | Provides monitoring/log collection resources | `Microsoft.OperationalInsights` was identified and registered before deployment. |
| Azure Local and Azure Arc integrations | Provide the Azure Local cluster and Azure management-plane experience required by the PoC | Required resource providers were validated as part of subscription readiness. |
| Resource Bridge / extended location capabilities | Support Arc and Azure Local resource projection/integration | Supporting resource providers were validated before deployment. |

### 2.3 Microsoft Validation and Engineering Controls

The Microsoft repository also contains engineering and integration-testing controls used by Microsoft to validate LocalBox. These are useful implementation references, but they are not automatically dependencies of our accreditation deployment.

| Microsoft validation control | Purpose in Microsoft engineering | Accreditation treatment |
| --- | --- | --- |
| Azure DevOps pipeline | Automates Microsoft LocalBox deployment and test execution | Reference only. We are not required to run Microsoft's internal pipeline to perform the accreditation PoC. |
| `New-AzResourceGroupDeployment` with `TemplateParameterObject` | Microsoft pipeline deployment pattern for passing Bicep parameters | Used as a supported-pattern reference while designing our secure local deployment flow. `Az.Resources 10.1.0` is now installed locally to support this deployment pattern. |
| Secure pipeline variable for Windows administrator password | Keeps the administrator password out of source code | Security pattern to follow. The actual password must remain runtime-only in our implementation. |
| Pester integration tests | Automated validation of deployed LocalBox behavior | Microsoft engineering validation mechanism. Not classified as a mandatory accreditation dependency unless we explicitly decide to run equivalent tests. |
| Automated teardown | Removes Microsoft test resource groups after validation or failure | Useful operational reference for cost control and cleanup, but not itself a prerequisite for deployment. |
| Scheduled pipeline runs | Repeatedly validate current Microsoft LocalBox source | Microsoft CI practice, not a deployment requirement for our accreditation lab. |

### Important distinction for presentation

Do not present every Microsoft engineering component as a mandatory customer or accreditation dependency.

Use this distinction:

- **Our Deployment Dependencies:** what our environment must have before deployment.
- **Microsoft LocalBox Solution Components:** what the LocalBox solution itself uses.
- **Microsoft Validation and Engineering Controls:** how Microsoft validates and maintains the solution internally.

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

#### Azure PowerShell deployment module

Microsoft's LocalBox validation pipeline uses `New-AzResourceGroupDeployment` with a parameter object. Local availability was checked and `Az.Resources` was initially absent. It was installed for the current user and then verified as:

```text
Az.Resources 10.1.0
```

Status: **PASS**.

---

## 4. Execution Continuity Dependency

Cloud Shell was initially used for readiness and source validation. During implementation it became clear that an ephemeral shell session is not ideal for work performed in multiple short sessions.

The execution model was therefore moved to a persistent VS Code workflow.

Verified local components:

- Git `2.55.0`.
- PowerShell `7.6.5`.
- Azure CLI `2.89.1`.
- Bicep CLI `0.46.1`.
- Az.Resources PowerShell module `10.1.0`.
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

The Microsoft LocalBox Bicep template marks the Windows administrator password as a secure parameter. Microsoft's own validation pipeline passes the value through a deployment parameter object rather than embedding the secret into the Bicep source. Our deployment method should preserve the same security objective even if the local execution mechanism differs from Microsoft's internal CI pipeline.

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
- Az.Resources PowerShell module `10.1.0`.
- Official Microsoft source pinning.
- Runtime artifact exact-commit reachability.
- Persistent VS Code execution environment.
- Resume workflow.

### Pending before billable deployment

1. Validate the local secure password-handling/deployment mechanism.
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
6. Microsoft solution components and Microsoft internal validation controls should not be incorrectly represented as customer-side mandatory dependencies.
7. Turning hidden prerequisites into a dependency register makes implementation delays explainable as controlled risk reduction rather than unstructured troubleshooting.

---

## 8. Presentation Content

This document must feed the accreditation presentation.

### Suggested slide: Implementation Readiness Controls

- Validated company subscription, Owner access, Central India region, quota, and VM SKU availability.
- Validated and registered required Azure Local, Arc, Compute, Network, Storage, Monitoring, and Log Analytics resource providers.
- Validated local deployment tooling including Azure CLI, Bicep, PowerShell 7, and `Az.Resources 10.1.0`.
- Moved execution from Cloud Shell to a persistent VS Code workflow to avoid session loss.
- Isolated the accreditation Azure CLI profile from other Azure projects on the workstation.
- Pinned the official Microsoft LocalBox source and validated runtime artifact reachability.

### Suggested slide: Microsoft LocalBox Implementation Reference

- Official LocalBox source is maintained in Microsoft's `azure_arc` repository.
- LocalBox uses Bicep for infrastructure deployment and PowerShell for supporting automation.
- Runtime scripts and artifacts can be retrieved from GitHub and therefore require source-reference control for repeatability.
- Microsoft LocalBox integrates Azure Compute, Networking, Storage, Log Analytics, Azure Local, and Azure Arc-related services.

### Suggested slide: Microsoft Validation vs Our Deployment Dependencies

- Microsoft uses an Azure DevOps pipeline to automate LocalBox deployment and integration testing.
- Microsoft's pipeline uses Azure PowerShell deployment commands and parameter objects, including secure handling of the administrator password.
- Microsoft also uses Pester integration tests and automated teardown in its validation process.
- These Microsoft engineering controls are useful implementation references, but they are not automatically mandatory dependencies of our accreditation PoC.

### Suggested slide: Dependency Register and Risk Reduction

- Converted implementation prerequisites into a formal dependency register.
- Identified `Microsoft.Network` and `Microsoft.OperationalInsights` before billable deployment.
- Prevented source drift by validating both local source pinning and runtime GitHub artifact references.
- Kept secrets and sensitive identifiers outside the public evidence repository.

### Suggested slide: Current Build Readiness

- Subscription, access, region, quota, tooling, source, runtime references, and direct provider dependencies: **Ready**.
- Pending before deployment: secure password handling and final deployment command validation.
- LocalBox workload deployment: **Not Started**.

---

## 9. Governance

This dependency register supports Accreditation Activity 3: Azure Local Proof of Concept using Microsoft Azure Arc Jumpstart LocalBox.

It does not replace the accreditation source-of-truth document. All implementation and presentation decisions must remain traceable to the required accreditation activities and customer scenario.