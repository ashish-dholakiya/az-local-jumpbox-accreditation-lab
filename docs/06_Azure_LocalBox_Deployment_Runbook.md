# Azure LocalBox Deployment Runbook

## Purpose

This runbook provides a clean, reproducible process for deploying Microsoft Azure Arc Jumpstart LocalBox and progressing through the validated Azure Local accreditation proof-of-concept workflow.

It is written for an engineer who wants to reproduce the lab without relying on undocumented assumptions.

Validated baseline:

```text
Azure region                  australiaeast
Azure Local instance region   australiaeast
LocalBox host VM size         Standard_E32s_v5
LocalBox host VM              LocalBox-Client
Azure Local cluster           localboxcluster
Microsoft source commit       3f433866757688d926ae6707e9c0041d8e640b82
```

Never store subscription IDs, tenant IDs, service-principal object IDs, passwords, tokens, credentials, or unsanitized screenshots in public documentation.

---

## 1. Understand the LocalBox Architecture

LocalBox deploys one outer Azure IaaS VM named `LocalBox-Client`.

Inside that VM, Hyper-V runs the nested LocalBox environment:

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

LocalBox is a lab and sandbox implementation. It is not the production Azure Local hardware model. Production Azure Local runs on supported and validated physical hardware.

Microsoft references:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

---

## 2. Prepare the Workstation

Validated tooling:

```text
PowerShell 7.6.5
Git 2.55.0.windows.5
Azure CLI 2.89.1
Bicep CLI 0.46.1
Az.Accounts 5.5.2
Az.Resources 10.2.0
```

Verify:

```powershell
$PSVersionTable.PSVersion
git --version
az version
az bicep version
```

Operational Azure Local validation also requires these Azure CLI extensions:

```text
customlocation
stack-hci-vm
```

Install explicitly when required:

```powershell
az extension add --name customlocation
az extension add --name stack-hci-vm
```

The pinned Microsoft `Configure-VMLogicalNetwork.ps1` also installs both extensions before performing Azure Local workload-network operations.

---

## 3. Clone and Pin the Official Microsoft Source

```powershell
Set-Location "C:\Projects"
git clone https://github.com/microsoft/azure_arc.git
Set-Location "C:\Projects\azure_arc"
git checkout 3f433866757688d926ae6707e9c0041d8e640b82
git status
git rev-parse HEAD
```

Required commit:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

The LocalBox Bicep directory is:

```text
C:\Projects\azure_arc\azure_jumpstart_localbox\bicep
```

Do not copy only `main.bicep` to another directory. LocalBox is a multi-module project with relative module dependencies.

---

## 4. Authenticate to Azure

```powershell
az login
az account set --subscription "<SUBSCRIPTION-NAME>"
```

Verify without exposing IDs:

```powershell
az account show `
  --query "{Subscription:name,State:state,IsDefault:isDefault}" `
  --output table
```

For Azure PowerShell operations:

```powershell
Disable-AzContextAutosave -Scope Process | Out-Null
Connect-AzAccount
```

---

## 5. Verify Mandatory Resource Providers

```powershell
$providers = @(
    "Microsoft.KubernetesConfiguration",
    "Microsoft.ExtendedLocation",
    "Microsoft.HybridContainerService",
    "Microsoft.HybridCompute",
    "Microsoft.AzureStackHCI",
    "Microsoft.ResourceConnector",
    "Microsoft.Kubernetes",
    "Microsoft.EdgeMarketplace"
)

foreach ($provider in $providers) {
    $state = az provider show `
        --namespace $provider `
        --query "registrationState" `
        --output tsv
    Write-Host "$provider : $state"
}
```

Every mandatory provider should report:

```text
Registered
```

LocalBox can request missing registration automatically, but registration is asynchronous. Verify readiness before cluster validation instead of depending only on the LocalBox polling window.

---

## 6. Use the Validated Australia East Configuration

| Parameter | Value |
| --- | --- |
| Resource group | `rg-azlocal-localbox-accreditation-aue` |
| Azure resource location | `australiaeast` |
| Azure Local instance location | `australiaeast` |
| LocalBox host VM size | `Standard_E32s_v5` |
| LocalBox host VM | `LocalBox-Client` |
| Deploy Bastion | `false` |
| Auto-deploy cluster resource | `true` |
| Auto-upgrade cluster resource | `false` |
| Spot pricing | `false` |
| VM auto-logon | `true` |
| Runtime source | exact pinned Microsoft commit |

Use the exact commit as the LocalBox runtime `githubBranch` value.

---

## 7. Create the Resource Group

```powershell
az group create `
  --name "rg-azlocal-localbox-accreditation-aue" `
  --location "australiaeast"
```

Verify:

```powershell
az group show `
  --name "rg-azlocal-localbox-accreditation-aue" `
  --query "{Name:name,Location:location,State:properties.provisioningState}" `
  --output table
```

---

## 8. Prepare Secure Runtime Inputs

Retrieve sensitive IDs only into process memory and do not print them into public evidence.

Example:

```powershell
$tenantId = az account show --query tenantId -o tsv
```

Capture passwords securely and do not save or print them.

---

## 9. Validate and Deploy LocalBox

Use the official Microsoft Bicep structure and the validated parameters.

A top-level ARM validation can report a nested-evaluation diagnostic such as `NestedDeploymentShortCircuited` when outputs from earlier modules are not yet available. Do not treat that diagnostic as full nested execution validation.

Successful host deployment creates resources including:

```text
LocalBox-Client
LocalBox-Client-OSDisk
LocalBox-Client-DataDisk_0 ... DataDisk_7
LocalBox-Client-NIC
LocalBox-Client-PIP
LocalBox-VNet
LocalBox-NSG
Log Analytics workspace
staging storage account
Bootstrap VM extension
```

Do not stop or deallocate `LocalBox-Client` while LocalBox automation is running.

---

## 10. Verify Bootstrap and Hyper-V

Useful guest checks include Hyper-V state, LocalBox scheduled-task state, and logs under:

```text
C:\LocalBox\Logs
```

Important logs include:

```text
Bootstrap.log
LocalBoxLogonScript.log
New-LocalBoxCluster.log
```

Expected build readiness:

```text
Hyper-V              Installed
LocalBoxLogonScript  Running
```

---

## 11. Monitor the Nested LocalBox Build

The pinned `New-LocalBoxCluster.ps1` workflow performs the major LocalBox build stages, including nested VM creation, networking and storage configuration, Azure Local cloud deployment preparation, validation, and final deployment.

Pinned source:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

---

## 12. Validate the Azure Local Cluster

Validation deployment:

```text
localcluster-validate
```

Check state:

```powershell
az deployment group show `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "localcluster-validate" `
  --query "{Name:name,State:properties.provisioningState,Error:properties.error}" `
  --output json
```

Required result:

```text
State : Succeeded
```

If provider readiness is the only blocker, correct the provider state and retry the narrow validation step rather than redeploying the complete lab.

---

## 13. Deploy and Monitor the Azure Local Cluster

Cluster deployment:

```text
localcluster-deploy
```

The validated lab deployment completed in approximately 2 hours 19 minutes.

A synchronous VM Run Command wrapper can time out while the ARM deployment continues. Monitor the authoritative ARM deployment:

```powershell
az deployment group show `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "localcluster-deploy" `
  --query "{State:properties.provisioningState,Duration:properties.duration,Error:properties.error}" `
  --output json
```

Do not rerun the deployment only because the initiating shell wrapper timed out.

Required final state:

```text
State : Succeeded
Error : null
```

---

## 14. Verify Azure Local Connectivity

```powershell
az resource show `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --resource-type "Microsoft.AzureStackHCI/clusters" `
  --name "localboxcluster" `
  --query "{ProvisioningState:properties.provisioningState,ConnectivityStatus:properties.connectivityStatus,Status:properties.status}" `
  --output json
```

Validated end state:

```text
ProvisioningState  : Succeeded
ConnectivityStatus : Connected
Status             : ConnectedRecently
```

Do not treat provisioning success alone as operational completion. Connectivity is a separate gate.

---

## 15. Understand the LocalBox Network Layers Before Creating Workloads

The validated network map is:

| Network | Purpose |
| --- | --- |
| `172.16.1.0/24` | Outer Azure subnet for `LocalBox-Client` |
| `192.168.1.0/24` | Azure Local infrastructure/management network |
| `192.168.128.0/24` | LocalBox host NAT transport network |
| `192.168.46.0/24` | LocalBox NAT/router routing subnet |
| `192.168.200.0/24` | Azure Local workload VM network, VLAN 200 |

The pinned Microsoft configuration explicitly defines:

```text
natHostSubnet = 192.168.128.0/24
natSubnet     = 192.168.46.0/24
vmGateway     = 192.168.200.1
vmIpPrefix    = 192.168.200.0/24
vmDNS         = 192.168.1.254
vmVLAN        = 200
```

Do not use the NAT transport or infrastructure subnet as the workload VM network.

---

## 16. Discover the Azure Local Custom Location

```powershell
az customlocation list `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --output table
```

Validated custom location:

```text
jumpstart
```

---

## 17. Verify Workload Storage Paths

Before VM lifecycle work, verify the Azure Local storage paths using the `stack-hci-vm` extension.

The validated lab contained two healthy workload storage paths under `C:\ClusterStorage`.

For this accreditation workflow, storage placement can be left automatic unless a specific workload requirement requires explicit affinity or storage selection.

---

## 18. Create the Workload Logical Network

Pinned Microsoft automation source:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/Configure-VMLogicalNetwork.ps1

The official LocalBox values are:

```text
Name       localbox-vm-lnet-vlan200
VM switch  ConvergedSwitch(compute_management)
Prefix     192.168.200.0/24
Gateway    192.168.200.1
DNS        192.168.1.254
VLAN       200
Allocation static
```

### GUI-first accreditation workflow

In Azure Portal, open the deployed Azure Local system and create a logical network using those values.

The validated lab left the explicit IP-pool fields blank because the pinned Microsoft script does not specify a manual pool.

### CLI verification

```powershell
az stack-hci-vm network lnet show `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "localbox-vm-lnet-vlan200" `
  --query "{Name:name,State:properties.provisioningState,Type:properties.networkType,VMSwitch:properties.vmSwitchName,DNS:properties.dhcpOptions.dnsServers,Subnets:properties.subnets}" `
  --output json
```

Validated state:

```text
ProvisioningState : Succeeded
Prefix            : 192.168.200.0/24
Gateway           : 192.168.200.1
DNS               : 192.168.1.254
VLAN              : 200
```

Backend inspection showed that the service materialized an IP pool spanning the workload subnet even though an explicit portal pool was not entered.

Status: **PASS**.

---

## 19. Add a Marketplace VM Image

For the first Windows workload, the validated portal selection was:

```text
[smalldisk] Windows Server 2025 Datacenter: Azure Edition - Gen2
```

Image resource name:

```text
ws2025-azureedition-smalldisk
```

Use:

```text
Custom location : jumpstart
Storage path    : Choose automatically
OS type         : Windows
VM generation   : Gen2
```

Automatic storage placement is appropriate for this lab because there is no requirement to force a specific storage path.

### Monitor backend progress

Do not depend only on the portal spinner. Use:

```powershell
az stack-hci-vm image show `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "ws2025-azureedition-smalldisk" `
  --query "{State:properties.provisioningState,Progress:properties.status.progressPercentage,DownloadMB:properties.status.downloadStatus.downloadSizeInMB,Operation:properties.status.provisioningStatus.status,ErrorCode:properties.status.errorCode,ErrorMessage:properties.status.errorMessage}" `
  --output json
```

Validated final result:

```text
Progress     : 100
State        : Succeeded
Operation    : Succeeded
ErrorCode    : empty
ErrorMessage : empty
```

### Long-running image observation

The Windows Server 2025 Marketplace image took roughly three hours to ingest in the nested LocalBox lab. Progress continued and no backend error was reported.

Treat this as a LocalBox lab observation only, not a production Azure Local performance benchmark.

Production image operations should maintain approved, reusable, versioned gold images rather than make image ingestion an on-demand prerequisite for every VM deployment.

Status: **PASS**.

---

## 20. Next Step: Create the First Azure Local VM

At this checkpoint the required prerequisites are ready:

```text
Azure Local cluster                 PASS
Azure connectivity                  PASS
Custom location                     PASS
Workload storage                    PASS
Workload logical network            PASS
Windows Server 2025 VM image        PASS
```

The next controlled step is to create one Azure Local VM through Azure Portal using:

```text
Image    ws2025-azureedition-smalldisk
Network  localbox-vm-lnet-vlan200
```

After VM creation, validate backend provisioning and then exercise start, stop, restart, and management operations before marking VM lifecycle complete.

---

## 21. Troubleshooting Rules

1. Verify provider registration before cluster validation.
2. Check `ConnectivityStatus` separately from `ProvisioningState`.
3. Inspect ARM operations before changing configuration.
4. A VM Run Command timeout is not automatically an Azure Local deployment failure.
5. Do not stop or deallocate `LocalBox-Client` during active LocalBox automation.
6. Keep Microsoft source pinned and unmodified unless a justified change is explicitly required.
7. Keep secrets and sensitive IDs out of public evidence.
8. Treat `customlocation` and `stack-hci-vm` as explicit workstation dependencies.
9. Distinguish outer Azure, infrastructure, NAT transport, routing, and workload networks.
10. Use the pinned LocalBox configuration as the source of truth for LocalBox-specific workload network values.
11. If a Marketplace image shows increasing progress and no error, do not delete or redeploy it simply because ingestion is slow.
12. Do not use nested LocalBox performance as a production Azure Local performance benchmark.

---

## 22. Quick Reference URLs

Microsoft Azure Arc Jumpstart repository:

https://github.com/microsoft/azure_arc

Pinned LocalBox host definition:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

Pinned LocalBox cluster automation:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

Pinned LocalBox configuration:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

Pinned workload logical-network automation:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/Configure-VMLogicalNetwork.ps1

---

## Current Verified Checkpoint

```text
Workstation tooling                  PASS
Official Microsoft source cloned     PASS
Microsoft source pinned              PASS
Azure authentication                 PASS
Australia East configuration         PASS
Mandatory provider readiness         PASS
LocalBox host deployment             PASS
Hyper-V readiness                    PASS
Nested LocalBox environment          PASS
Arc integration                      PASS
localcluster-validate                Succeeded
localcluster-deploy                  Succeeded
Azure Local cluster connectivity     Connected
Azure Local cluster status           ConnectedRecently
customlocation extension             PASS
stack-hci-vm extension               PASS
Logical workload network             PASS
Marketplace VM image                 PASS
Azure Local VM lifecycle             PENDING
Monitoring and management            PENDING
Update and lifecycle validation      PENDING
```
