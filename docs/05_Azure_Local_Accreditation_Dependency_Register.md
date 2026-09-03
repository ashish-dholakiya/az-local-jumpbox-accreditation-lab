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
| Azure CLI `customlocation` extension | Required by `az customlocation` commands used for Azure Local custom-location discovery and validation | PASS |
| Azure CLI `stack-hci-vm` extension | Required by `az stack-hci-vm` commands used for logical network, image, and VM lifecycle operations | PASS |
| Bicep CLI 0.46.1 | LocalBox Bicep deployment workflow | PASS |
| Az.Accounts 5.5.2 | Azure PowerShell authentication | PASS |
| Az.Resources 10.2.0 | ARM validation and deployment operations | PASS |
| Separate Microsoft repository | Keeps official source isolated from project documentation | PASS |
| Exact Microsoft commit pin | Prevents source drift | PASS |
| Runtime `githubBranch` pinned to exact commit | Prevents runtime artifact drift | PASS |

### Azure CLI extension behavior

The first execution of a command whose extension is not installed can produce an interactive installation prompt. This was observed with both `customlocation` and `stack-hci-vm` during the operational validation phase.

Installing these extensions is a **local Azure CLI tooling change only**. It does not create, modify, or delete Azure Local or Azure subscription resources.

The pinned Microsoft `Configure-VMLogicalNetwork.ps1` also explicitly installs both extensions before creating the LocalBox workload logical network:

```powershell
az extension add --name customlocation
az extension add --name stack-hci-vm
```

For unattended automation, extension installation should be configured deliberately rather than depending on an interactive prompt.

Status: **PASS**.

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

## 11. Layered Networking Dependency

LocalBox contains several distinct network layers. Treating one of the transport networks as the workload network would produce incorrect VM networking.

Verified network map:

| Network | Function | Status |
| --- | --- | --- |
| `172.16.1.0/24` | Outer Azure subnet for `LocalBox-Client` | PASS |
| `192.168.1.0/24` | Azure Local infrastructure/management network | PASS |
| `192.168.128.0/24` | LocalBox host NAT transport network | PASS |
| `192.168.46.0/24` | LocalBox NAT/router routing subnet | PASS |
| `192.168.200.0/24` | Azure Local workload VM network, VLAN 200 | PASS |

The pinned Microsoft configuration defines:

```text
natHostSubnet = 192.168.128.0/24
natSubnet     = 192.168.46.0/24
vmGateway     = 192.168.200.1
vmIpPrefix    = 192.168.200.0/24
vmDNS         = 192.168.1.254
vmVLAN        = 200
```

This confirms that `192.168.128.0/24` and `192.168.46.0/24` are LocalBox transport/routing networks, not the Azure Local workload subnet.

Status: **PASS**.

---

## 12. Logical Workload Network Dependency

The pinned Microsoft `Configure-VMLogicalNetwork.ps1` defines the expected workload logical network as:

```text
Name       localbox-vm-lnet-vlan200
VM switch  ConvergedSwitch(compute_management)
Prefix     192.168.200.0/24
Gateway    192.168.200.1
DNS        192.168.1.254
VLAN       200
Allocation static
```

The logical network was created through Azure Portal using those values and was then validated through `az stack-hci-vm network lnet show`.

Final provisioning state:

```text
Succeeded
```

The portal workflow did not require an explicit IP pool. Backend inspection showed that the service materialized a pool spanning the workload subnet after creation.

Status: **PASS**.

---

## 13. Workload Storage Dependency

Two Azure Local workload storage paths were discovered and validated before VM lifecycle work:

```text
UserStorage1-62e88607ab3c4e25b3dbe4a59b0586c3
UserStorage2-96458537fbbd4688a9cb1745f822f5d2
```

Both reported `Succeeded` and mapped to cluster storage paths under `C:\ClusterStorage`.

For the Marketplace image exercise, Azure Portal storage placement was left on `Choose automatically`. This was intentional because the lab has no requirement to force image placement to a specific storage path.

Status: **PASS**.

---

## 14. Azure Local Marketplace Image Dependency

The first workload VM image was created from Azure Marketplace through Azure Portal:

```text
[smalldisk] Windows Server 2025 Datacenter: Azure Edition - Gen2
```

Image resource name:

```text
ws2025-azureedition-smalldisk
```

Final backend validation:

```text
Progress     : 100
State        : Succeeded
Operation    : Succeeded
ErrorCode    : empty
ErrorMessage : empty
```

### Long-running image ingestion observation

The image ingestion took roughly three hours in this nested LocalBox environment. Progress continued throughout the operation and the backend reported no error.

This is a **lab observation, not a production benchmark**. LocalBox includes nested virtualization, virtualized networking, routing/NAT, and virtualized storage layers that can materially affect ingestion behavior.

For production operations, approved gold images should be prepared, patched, tested, versioned, published, reused, and retired through a defined image lifecycle. Image download/import should not be an on-demand prerequisite for every VM deployment.

Status: **PASS**.

---

## 15. Security Controls

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

## 16. Current Dependency Readiness

Status synchronized on **3 September 2026** with the [VM network, guest-management and lifecycle evidence](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md). PASS refers to the recorded validation; it does not represent a new test in this documentation update. Workload VM start/stop/restart is complete; Azure Local platform update/lifecycle validation is a separate pending activity.

| Area | Status |
| --- | --- |
| Subscription and access | PASS |
| Australia East region | PASS |
| VM SKU | PASS |
| Workstation tooling | PASS |
| Azure CLI `customlocation` extension | PASS |
| Azure CLI `stack-hci-vm` extension | PASS |
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
| Logical workload networking | PASS |
| Workload storage readiness | PASS |
| Azure Local Marketplace VM image | PASS |
| Azure Local workload VM creation and start/stop/restart | PASS |
| Azure monitoring and management | NEXT / pending validation |
| Azure Local platform update and lifecycle validation | PENDING |

---


Next: verify the `AzureEdgeTelemetryAndDiagnostics` extension prerequisite before collecting the remaining monitoring and management evidence.

### Cloud Witness and storage-access dependencies (3 September 2026)

The startup recovery exposed ongoing dependencies beyond initial deployment success:

| Dependency | Why it matters | Verified checkpoint / follow-up |
| --- | --- | --- |
| Witness configured in cluster quorum | An Online Cluster Group alone does not establish that a witness exists | Cloud Witness configured and Online |
| Storage authentication permitted | The deployed key-based witness requires usable storage authentication | Access restored; witness configuration succeeded with the existing key |
| Storage data-plane network path | Authentication settings alone do not provide network access | Recovery succeeded after restoring the intended selected-network access |
| Governance compatibility | Subsequent policy or automation changes can affect required access | Temporary workaround; permanent resolution pending |
| Exception expiry ownership | A reminder does not renew an exception | Follow-up and expiry recorded in the linked checkpoint |
| Node and CSV readiness after startup | Node availability, volume health and witness health are separate checks | Both nodes Up; all three CSVs Online |

The pinned [LocalBox deployment template](https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/azlocal.json) defines the witness storage account and storage-key secret used by cluster deployment. Its [parameter template](https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/azlocal.parameters.json) selects Cloud witness.

See the [startup and Cloud Witness runbook](06_Azure_LocalBox_Deployment_Runbook.md#211-resume-an-existing-lab-and-troubleshoot-cloud-witness) and [dated recovery evidence and follow-up](10_Lab_Recovery_Checkpoint_and_Follow_Up.md). These are point-in-time recovery results, not a claim that long-term governance compatibility is resolved.

## 17. Presentation Positioning

Recommended message:

> We treated subscription access, provider readiness, source pinning, regional compute availability, nested virtualization, secure runtime inputs, deployment validation, workload networking, image readiness, and operational connectivity as explicit implementation dependencies. This converted troubleshooting into controlled readiness validation and produced a repeatable Azure Local PoC process.

Key presentation lessons:

1. Validate providers before cluster validation.
2. Distinguish outer Azure VM deployment from nested Azure Local nodes.
3. Monitor long-running Azure Local deployments through Azure Resource Manager rather than only through the initiating shell command.
4. Treat `ProvisioningState = Succeeded` and `ConnectivityStatus = Connected` as separate completion gates.
5. Keep Microsoft source pinned and secrets out of public evidence.
6. Treat Azure CLI extensions such as `customlocation` and `stack-hci-vm` as explicit workstation dependencies.
7. Distinguish LocalBox transport networks from the Azure Local workload logical network.
8. Maintain approved, reusable, versioned gold images in production rather than downloading an image for every VM deployment.
9. Do not use LocalBox image-ingestion timing as a production Azure Local performance benchmark.
