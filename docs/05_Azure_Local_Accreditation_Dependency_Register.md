# Azure Local Accreditation Lab Dependency Register

## Purpose

This document records the dependencies that were required, discovered, validated, and remediated during the Azure Local accreditation lab using Microsoft Azure Arc Jumpstart LocalBox.

The register supports repeatability, troubleshooting, implementation evidence, and presentation preparation.

The active lab baseline is Australia East only.

---

## 1. Verified Deployment Baseline

```text
Azure region                  australiaeast
Azure Local instance region   australiaeast
LocalBox host VM size         Standard_E32s_v5
LocalBox host VM              LocalBox-Client
Microsoft source commit       3f433866757688d926ae6707e9c0041d8e640b82
Azure Local cluster           localboxcluster
```

Sensitive identifiers and secrets are intentionally excluded.

---

## 2. Workstation and Execution Dependencies

| Dependency | Why it matters | Verified state |
| --- | --- | --- |
| PowerShell 7.6.5 | Controlled PowerShell execution environment | PASS |
| Git 2.55.0.windows.5 | Clone and pin Microsoft source | PASS |
| Azure CLI 2.89.1 | Authentication, provider checks, resource and deployment queries | PASS |
| Azure CLI `customlocation` extension | Required by `az customlocation` commands used to discover and validate the Azure Local custom location before VM lifecycle work | DISCOVERED / INSTALLING |
| Bicep CLI 0.46.1 | LocalBox Bicep deployment workflow | PASS |
| Az.Accounts 5.5.2 | Azure PowerShell authentication | PASS |
| Az.Resources 10.2.0 | ARM validation and deployment operations | PASS |
| Separate Microsoft repository | Keeps official source isolated from project documentation | PASS |
| Exact Microsoft commit pin | Prevents source drift | PASS |
| Runtime `githubBranch` pinned to exact commit | Prevents runtime artifact drift | PASS |

### Azure CLI `customlocation` extension behavior

The first execution of an `az customlocation` command on a workstation where the extension is not already installed can produce an interactive prompt similar to:

```text
The command requires the extension customlocation.
Do you want to install it now? (Y/n)
```

Azure CLI can then search for and install the required extension before continuing the original command.

This is a **local Azure CLI tooling change only**. Installing the extension does not create, modify, or delete Azure Local or Azure subscription resources.

Azure CLI may also display guidance about dynamic extension installation and preview-version behavior. For unattended automation, extension installation behavior should be configured deliberately rather than relying on an interactive prompt. For this accreditation lab, the interactive installation was accepted when first encountered, and the resulting extension state should be verified after installation completes.

This dependency is important for repeatability because a fresh workstation can otherwise appear to fail at the custom-location discovery step even though the Azure Local environment itself is healthy.

---

## 3. Azure Subscription and Access Dependencies

| Dependency | Why it matters | Verified state |
| --- | --- | --- |
| Correct accreditation subscription | Prevents deployment to the wrong Azure environment | PASS |
| Effective Owner-level access | Required for provider registration and deployment operations | PASS |
| Australia East resource group | Dedicated deployment and cleanup boundary | PASS |
| `Standard_E32s_v5` usability | Required for the LocalBox outer host VM | PASS |

---

## 4. Mandatory Azure Local Resource Providers

The LocalBox workflow and Azure Local Environment Validator require the following providers for cluster integration:

```text
Microsoft.KubernetesConfiguration
Microsoft.ExtendedLocation
Microsoft.HybridContainerService
Microsoft.HybridCompute
Microsoft.AzureStackHCI
Microsoft.ResourceConnector
Microsoft.Kubernetes
Microsoft.EdgeMarketplace
```

Final verified state:

```text
Microsoft.KubernetesConfiguration : Registered
Microsoft.ExtendedLocation        : Registered
Microsoft.HybridContainerService  : Registered
Microsoft.HybridCompute           : Registered
Microsoft.AzureStackHCI           : Registered
Microsoft.ResourceConnector       : Registered
Microsoft.Kubernetes              : Registered
Microsoft.EdgeMarketplace         : Registered
```

### Important implementation lesson

LocalBox attempts to register missing providers automatically, but Azure provider registration is asynchronous. The LocalBox polling window can end before Azure has fully propagated registration.

During this lab, Azure Local Arc Integration validation detected that `Microsoft.HybridContainerService` had not yet completed registration. `Microsoft.HybridContainerService` and `Microsoft.EdgeMarketplace` were explicitly registered and monitored until both reached `Registered`, after which validation succeeded.

This dependency should therefore be checked before cluster validation rather than relying only on the automatic LocalBox registration request.

---

## 5. Direct Azure Resource Dependencies

The official LocalBox Bicep deployment also uses Azure resource providers and services including:

```text
Microsoft.Authorization
Microsoft.Compute
Microsoft.Network
Microsoft.OperationalInsights
Microsoft.Resources
Microsoft.Storage
Microsoft.KeyVault
Microsoft.Insights
```

Relevant providers were verified and registered before or during implementation.

---

## 6. LocalBox Architecture Dependency

The lab depends on nested virtualization.

One Azure IaaS VM is deployed:

```text
LocalBox-Client
```

Inside it, Hyper-V hosts the nested LocalBox environment:

```text
AzLMGMT
AzLHOST1
AzLHOST2
```

`AzLHOST1` and `AzLHOST2` are the two Azure Local nodes. They are not separate top-level Azure IaaS VMs.

### Why this matters

LocalBox is a lab/sandbox implementation. The outer Azure VM provides the compute, storage, networking, and Hyper-V capability required to emulate Azure Local without requiring separate physical Azure Local servers.

This is not the production Azure Local hardware model.

Microsoft references:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

---

## 7. Bootstrap and Continuation Dependencies

LocalBox host deployment alone is not the end of deployment.

The Bootstrap extension performs additional preparation, including Hyper-V installation and creation of the LocalBox logon workflow. Hyper-V requires reboot completion before the nested environment can continue.

Verified state:

```text
Bootstrap extension   Succeeded
Hyper-V               Installed
LocalBox automation   Started successfully
```

The `LocalBox-Client` VM must not be stopped or deallocated while this automation is still running.

---

## 8. Cluster Validation Dependency

The Azure Local validation deployment is:

```text
localcluster-validate
```

Validation must pass before the actual cluster deployment is considered ready.

Final result:

```text
localcluster-validate : Succeeded
```

The final validation report showed successful checks across Remote Management, Connectivity, Active Directory, Hardware, Networking, Observability, Software, MOC Stack, Arc Integration, Cluster Witness, DNS, NTP, Cluster Validation, LLDP, and Storage Validation.

---

## 9. Actual Cluster Deployment Dependency

The Azure Local cluster deployment is:

```text
localcluster-deploy
```

The deployment took approximately 2 hours 19 minutes.

Final authoritative state:

```text
State    : Succeeded
Error    : null
```

The child deployment settings also reported:

```text
provisioningState       : Succeeded
deploymentStatus.status : Success
validationStatus.status : Success
```

### Long-running deployment note

A synchronous Azure VM Run Command invocation can time out while the Azure Local ARM deployment continues successfully in the Azure control plane.

Therefore, long-running deployment health should be monitored using ARM deployment status and deployment operations rather than interpreting a Run Command timeout as Azure Local deployment failure.

---

## 10. Final Connectivity Dependency

Deployment success alone is not sufficient. Azure Local must also be operationally connected to Azure.

Final verified cluster state:

```text
ProvisioningState  : Succeeded
ConnectivityStatus : Connected
Status             : ConnectedRecently
```

Status: **PASS**.

---

## 11. Security Controls

The following must never be committed to the public repository:

- full Azure subscription ID
- tenant ID
- Azure Local resource-provider object ID
- Windows administrator password
- access tokens
- credentials or secrets
- sensitive screenshots

The LocalBox administrator password is runtime-only input.

---

## 12. Current Dependency Readiness

| Area | Status |
| --- | --- |
| Subscription and access | PASS |
| Australia East region | PASS |
| VM SKU | PASS |
| Workstation tooling | PASS |
| Azure CLI `customlocation` extension | DISCOVERED / INSTALLING |
| Microsoft source pinning | PASS |
| Runtime source pinning | PASS |
| Mandatory resource providers | PASS |
| LocalBox host deployment | PASS |
| Bootstrap and Hyper-V | PASS |
| Nested Azure Local nodes | PASS |
| Arc integration | PASS |
| Azure Local validation | PASS |
| Azure Local deployment | PASS |
| Azure Local connectivity | PASS |
| Azure Local VM lifecycle | PENDING |
| Logical workload networking | PENDING |
| Monitoring and management | PENDING |
| Update and lifecycle validation | PENDING |

---

## 13. Presentation Positioning

Recommended message:

> We treated subscription access, provider readiness, source pinning, regional compute availability, nested virtualization, secure runtime inputs, deployment validation, and operational connectivity as explicit implementation dependencies. This converted troubleshooting into controlled readiness validation and produced a repeatable Azure Local PoC deployment process.

Key presentation lessons:

1. Validate providers before cluster validation.
2. Distinguish outer Azure VM deployment from nested Azure Local nodes.
3. Monitor long-running Azure Local deployments through Azure Resource Manager rather than only through the initiating shell command.
4. Treat `ProvisioningState = Succeeded` and `ConnectivityStatus = Connected` as separate completion gates.
5. Keep Microsoft source pinned and secrets out of public evidence.
6. Treat Azure CLI extensions such as `customlocation` as explicit workstation dependencies for repeatable operational validation.
