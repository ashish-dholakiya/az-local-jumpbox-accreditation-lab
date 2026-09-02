# Azure LocalBox Lab Walkthrough

## Purpose

This document records the verified implementation steps for Accreditation Activity 3: Azure Local proof of concept using Microsoft Azure Arc Jumpstart LocalBox.

The governing scope and customer scenario remain in `docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md`.

This walkthrough records only actions that were actually executed and verified.

---

## 1. Official Microsoft LocalBox Source

The official Microsoft repository used for the lab is:

https://github.com/microsoft/azure_arc

The implementation is pinned to:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

Commit description:

```text
Update LocalBox to use image version 2608 (#3521)
```

LocalBox Bicep path:

```text
azure_jumpstart_localbox/bicep
```

The exact commit was also used as the LocalBox runtime `githubBranch` value so downloaded automation artifacts remain aligned to the tested source revision.

Status: **PASS**.

---

## 2. Validated Workstation Tooling

Verified execution baseline:

```text
PowerShell 7.6.5
Git 2.55.0.windows.5
Azure CLI 2.89.1
Bicep CLI 0.46.1
Az.Accounts 5.5.2
Az.Resources 10.2.0
```

The accreditation repository and Microsoft repository are maintained separately.

Status: **PASS**.

---

## 3. Australia East Deployment Configuration

The verified deployment configuration is:

| Parameter | Verified value |
| --- | --- |
| Resource group | `rg-azlocal-localbox-accreditation-aue` |
| Azure resource location | `australiaeast` |
| Azure Local instance location | `australiaeast` |
| LocalBox host VM size | `Standard_E32s_v5` |
| LocalBox host VM | `LocalBox-Client` |
| `deployBastion` | `false` |
| `autoDeployClusterResource` | `true` |
| `autoUpgradeClusterResource` | `false` |
| `enableAzureSpotPricing` | `false` |
| `vmAutologon` | `true` |
| Runtime source | exact pinned Microsoft commit |

Secrets, tenant identifiers, subscription identifiers, service-principal object identifiers, passwords, and tokens are intentionally excluded from this public repository.

Status: **PASS**.

---

## 4. LocalBox Architecture: One Azure VM, Nested Azure Local Nodes

LocalBox does not deploy two separate Azure IaaS VMs for the two Azure Local nodes.

The outer Azure resource is one VM:

```text
LocalBox-Client
```

Inside `LocalBox-Client`, Hyper-V provides nested virtualization and LocalBox creates:

```text
Azure Subscription
|
+-- LocalBox-Client
    Azure IaaS VM
    |
    +-- Hyper-V / Nested Virtualization
        |
        +-- AzLMGMT
        |   Management VM
        |
        +-- AzLHOST1
        |   Azure Local Node 1
        |
        +-- AzLHOST2
            Azure Local Node 2
```

### Why LocalBox uses this model

LocalBox is a lab and sandbox implementation. A single sufficiently large Azure VM supplies the compute, storage, networking, and nested virtualization capacity required to emulate an Azure Local environment. The Azure Local nodes are then created as nested Hyper-V VMs.

This allows engineers to exercise Azure Local deployment, Azure Arc integration, cluster validation, management, and lifecycle scenarios without requiring separate physical Azure Local servers for the lab.

This is not the production Azure Local deployment model. Production Azure Local runs on supported and validated physical Azure Local hardware.

### Microsoft source references

Outer Azure VM definition:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

Nested VM configuration:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

Nested Azure Local node creation:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

Status: **PASS**.

---

## 5. LocalBox Host Deployment

The LocalBox host deployment completed successfully in Australia East.

Verified Azure resources include:

- `LocalBox-Client`
- `LocalBox-Client-OSDisk`
- `LocalBox-Client-DataDisk_0` through `_7`
- `LocalBox-Client-NIC`
- `LocalBox-Client-PIP`
- `LocalBox-VNet`
- `LocalBox-NSG`
- Log Analytics workspace
- staging storage account
- `Bootstrap` VM extension

The host VM was verified running and the Bootstrap extension reached `Succeeded`.

Status: **PASS**.

---

## 6. Bootstrap, Reboot and Hyper-V Readiness

Bootstrap installed the required tooling and Hyper-V role. Hyper-V installation required a reboot before the LocalBox logon workflow could continue.

Post-reboot checks confirmed:

```text
Hyper-V InstallState : Installed
LocalBoxLogonScript   : Running
```

Relevant logs were created under:

```text
C:\LocalBox\Logs
```

including:

```text
Bootstrap.log
LocalBoxLogonScript.log
New-LocalBoxCluster.log
```

Status: **PASS**.

---

## 7. Nested Azure Local Environment Creation

`New-LocalBoxCluster.ps1` created and configured the nested LocalBox environment.

The Microsoft workflow includes eleven high-level stages:

1. Download LocalBox VHDs.
2. Prepare the Azure VM virtualization host.
3. Create the management VM.
4. Create Azure Local node VMs.
5. Start nested VMs.
6. Configure networking and storage.
7. Build the router VM.
8. Build the domain controller VM.
9. Prepare the Azure Local cluster cloud deployment.
10. Validate the Azure Local cluster deployment.
11. Deploy the Azure Local cluster.

Arc registration for the nested environment succeeded before final cluster deployment.

Status: **PASS**.

---

## 8. Mandatory Resource Provider Readiness

The LocalBox workflow checks these mandatory providers before Azure Local cluster validation:

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

LocalBox attempts to register missing providers automatically, but provider registration is asynchronous and can take longer than the script polling window.

During validation, `Microsoft.HybridContainerService` had not completed registration and Azure Local Arc Integration validation correctly failed.

The following providers were explicitly registered and monitored until they reached `Registered`:

```text
Microsoft.HybridContainerService
Microsoft.EdgeMarketplace
```

Final state for all mandatory providers: **Registered**.

Status: **PASS AFTER REMEDIATION**.

---

## 9. Azure Local Cluster Validation

The validation deployment is:

```text
localcluster-validate
```

The first strict Environment Validator run identified the incomplete resource-provider registration at the Azure Local Arc Integration step.

After provider readiness was corrected, only the failed validation was retried using the already-generated LocalBox ARM files:

```text
C:\LocalBox\azlocal.json
C:\LocalBox\azlocal.parameters.json
```

Verified result:

```text
DeploymentName    : localcluster-validate
ProvisioningState : Succeeded
```

The final validation status subsequently showed success across the Azure Local validation stages, including Remote Management, Connectivity, Active Directory, Hardware, Networking, Observability, Software, MOC Stack, Arc Integration, Cluster Witness, DNS, NTP, Cluster Validation, LLDP and Storage Validation.

Status: **PASS**.

---

## 10. Azure Local Cluster Deployment

The actual cluster deployment is:

```text
localcluster-deploy
```

The deployment was triggered using the generated LocalBox ARM template and parameter file.

The VM Run Command wrapper timed out while waiting for the long-running operation. That timeout did not represent Azure Local deployment failure. The authoritative Azure Resource Manager deployment continued independently.

Final outer deployment result:

```text
State    : Succeeded
Duration : approximately 2 hours 19 minutes
Error    : null
```

Direct `deploymentSettings/default` inspection also confirmed:

```text
provisioningState          : Succeeded
deploymentMode             : Deploy
deploymentStatus.status    : Success
validationStatus.status    : Success
```

Both Azure Local nodes were present as Arc-enabled machine resources:

```text
AzLHOST1
AzLHOST2
```

Status: **PASS**.

---

## 11. Final Azure Local Connectivity

Final Azure-side cluster state:

```text
ConnectivityStatus : Connected
ProvisioningState  : Succeeded
Status             : ConnectedRecently
```

This confirms that the Azure Local cluster is not only provisioned as an ARM resource but operationally connected to Azure.

Status: **PASS**.

---

## 12. Portal Validation Evidence

Azure Portal access was validated after the cluster deployment completed. The Azure Local resource was opened through the Azure Arc supported-environments experience and the deployed system was visible as `localboxcluster`.

Verified portal observations include:

- Azure Local system name: `localboxcluster`.
- Azure connection state: connected.
- Azure region: Australia East.
- Machine count: `2`.
- Azure Local updates state: `Up to date` at the time of capture.
- Cluster overview and management experience accessible from Azure Portal.
- Workload virtual machine count remained `0` at this checkpoint because VM lifecycle validation had not yet started.

### Sanitized portal evidence

![Azure Local cluster overview](../evidence/portal/azure-local-cluster-overview-redacted.png)

*Figure 1. Sanitized Azure Local cluster overview showing the deployed `localboxcluster` management experience without exposing subscription or account identifiers.*

![Azure Local cluster updates and management view](../evidence/portal/azure-local-cluster-overview-updates.png)

*Figure 2. Sanitized Azure Local portal evidence showing the current update and cluster management state.*

The source screenshots were sanitized before being committed to the public repository. Full subscription IDs, account identifiers, tenant identifiers, credentials, and other sensitive values are intentionally excluded.

Status: **PASS**.

---

## 13. Current Activity 3 Status

| Implementation checkpoint | Status |
| --- | --- |
| Azure subscription readiness | PASS |
| Official Microsoft source pinned | PASS |
| Australia East deployment configuration | PASS |
| Workstation tooling | PASS |
| Mandatory provider readiness | PASS |
| LocalBox host deployment | PASS |
| Bootstrap and Hyper-V | PASS |
| Nested management and Azure Local nodes | PASS |
| Arc registration | PASS |
| Azure Local cluster validation | PASS |
| Azure Local cluster deployment | PASS |
| Azure Local connectivity | PASS |
| Azure Portal cluster visibility | PASS |
| Portal validation evidence | PASS |
| Azure Local VM lifecycle validation | PENDING |
| Logical workload networking validation | PENDING |
| Azure monitoring and management validation | PENDING |
| Azure Local update and lifecycle validation | PENDING |

---

## Next Required Step

The LocalBox infrastructure and Azure Local cluster are successfully deployed, connected, and validated in Azure Portal.

The next implementation work should move to the remaining Accreditation Activity 3 operational outcomes, one at a time:

1. Azure Local VM creation and lifecycle.
2. Logical workload networking.
3. Azure monitoring and management.
4. Azure Local update and lifecycle validation.

Each outcome should be executed, verified, and recorded before being marked complete.
