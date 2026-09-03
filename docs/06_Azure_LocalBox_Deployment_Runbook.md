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

## 20. Create and Validate the First Azure Local VM

For a fresh lab, complete these prerequisites before creating the first workload VM:

```text
Azure Local cluster                 PASS
Azure connectivity                  PASS
Custom location                     PASS
Workload storage                    PASS
Workload logical network            PASS
Windows Server 2025 VM image        PASS
```

For a fresh lab, create one Azure Local VM through Azure Portal using:

```text
Image    ws2025-azureedition-smalldisk
Network  localbox-vm-lnet-vlan200
```

After VM creation, validate backend provisioning and then exercise start, stop, restart, and management operations before marking VM lifecycle complete.

**Existing workload VM stage: complete.** Workload VM creation, NIC/IP, guest management, and start/stop/restart are verified in [VM network, guest-management and lifecycle evidence](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md). This section remains a deployment procedure, not the current next task. Basic monitoring has subsequently been verified. Follow section 21.2 and the [monitoring and update evidence](11_Azure_Monitoring_and_Platform_Update_Validation.md) for the completed platform installation and verified post-update evidence.

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


### 21.1 Resume an Existing Lab and Troubleshoot Cloud Witness

This procedure captures the recovery verified on 3 September 2026. Use it when resuming the existing lab; do not restart deployment automation merely because the host was stopped. The [dated checkpoint](10_Lab_Recovery_Checkpoint_and_Follow_Up.md) contains the outcome and follow-up dates.

#### A. Confirm prerequisites and current state (read-only)

Before starting the lab session, verify the active governance exception and its expiry where applicable, permitted storage authentication, and the intended storage network access. A portal save or exception-creation notification is not a health check.

After the outer host and nested machines have started, run the following from an elevated PowerShell session directly on an Azure Local node:

```powershell
Get-ClusterNode |
    Format-Table Name, State -AutoSize

Get-ClusterQuorum |
    Format-List Cluster, QuorumType, QuorumResource

Get-ClusterResource |
    Where-Object { $_.ResourceType -like '*Witness*' } |
    Format-Table Name, State, ResourceType, OwnerNode -AutoSize

Get-ClusterGroup -Name 'Cluster Group' |
    Format-Table Name, State, OwnerNode -AutoSize

Get-ClusterSharedVolume |
    Format-Table Name, State, OwnerNode -AutoSize

Get-VirtualDisk |
    Format-Table FriendlyName, HealthStatus, OperationalStatus, DetachedReason -AutoSize

Get-StorageJob |
    Format-Table Name, JobState, PercentComplete -AutoSize
```

Capture actual state before changing anything. During the observed recovery, volumes became healthy while Cloud Witness remained failed. That sequence does not prove the witness caused the volume failures. Do not infer disk damage from a startup dashboard alone or change attachment policy solely because a disk reports Detached By Policy.

#### B. Diagnose the specific remaining failure

| Observed result | Interpretation and next check |
| --- | --- |
| Witness Online and QuorumResource populated | Witness is configured; no witness repair is needed |
| No witness resource and empty QuorumResource | No witness is configured; an Online Cluster Group does not close this gap |
| Witness Failed; event 1659 | Storage authentication/access failed; do not assume the key was rotated |
| HTTP 403 with AuthorizationFailure | Inspect storage network access and firewall restrictions as well as the full error |
| Shared-key access becomes disabled again | Inspect Activity Log changes and applicable policy; establish attribution before choosing remediation |
| Network settings disappear or fail to save | Inspect the save result and Activity Log; a Deny rejection and a later configuration reversal are different cases |
| Node, volume or storage job remains unhealthy | Investigate that layer separately; witness recovery is not proof of storage recovery |

For a failed witness, inspect recent Failover Clustering events on its current owner node. Run this locally on that node:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName      = 'System'
    ProviderName = 'Microsoft-Windows-FailoverClustering'
    Id           = 1069, 1659
    StartTime    = (Get-Date).AddMinutes(-30)
} |
    Select-Object TimeCreated, Id, Message |
    Format-List
```

Compare timestamps in UTC before diagnosing clock skew. Review error output locally and sanitize it before publication.

In Azure Portal, inspect the existing witness account:
- Configuration: permitted authentication, including Allow storage account key access for this deployed key-based witness.
- Networking: public-network mode, allowed networks, private endpoints and any network security perimeter.
- Activity Log: the relevant write or policy event and its actual changed property.

If the intended path uses selected virtual networks, confirm both the subnet's Microsoft.Storage service endpoint and the storage account's subnet rule. Public network access Disabled prevents this public-endpoint path. Do not widen access to all networks as a default troubleshooting step. See [Microsoft's network-rule guidance](https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security-virtual-networks) and [403 diagnostic guidance](https://learn.microsoft.com/en-us/troubleshoot/azure/azure-storage/blobs/authentication/storage-troubleshoot-403-errors).

#### C. Restore the approved access path (configuration change)

Resolve the identified authentication/network restriction through the appropriate governance process. Record scope, owner, expiry and follow-up for any temporary exception. A whole-initiative lab waiver is not a default production remediation, and does not disable separate assignments or independent automation.

Save the intended settings, reopen the portal page and verify that they persist. Allow for any propagation period reported by the service. Repeatedly re-enabling a setting while automation reverses it is not a durable fix.

#### D. Recover the witness only if required (configuration change)

If the witness exists but remains Failed after the access correction, attempt one controlled start:

```powershell
Start-ClusterResource -Name 'Cloud Witness' -Wait 60 -ErrorAction Stop
```

If it fails, inspect the new error rather than repeating the same start command.

If the witness is absent, or evidence requires refreshing its configured credentials, use the existing approved storage account and current key. The following is the same recovery command pattern that succeeded in this lab, with the account name prompted to keep environment identifiers out of the example. Run directly on a cluster node in elevated PowerShell.

```powershell
$witnessAccount = Read-Host 'Existing witness storage account name'
$witnessKey = Read-Host 'Existing witness storage account key' -AsSecureString

try {
    Set-ClusterQuorum `
        -CloudWitness `
        -AccountName $witnessAccount `
        -Endpoint 'core.windows.net' `
        -AccessKey ([System.Net.NetworkCredential]::new('', $witnessKey).Password) `
        -ErrorAction Stop | Out-Null

    Get-ClusterQuorum |
        Format-List Cluster, QuorumType, QuorumResource

    Get-ClusterResource -Name 'Cloud Witness' |
        Format-List Name, State, OwnerNode
}
catch {
    Write-Host "Cloud Witness configuration failed: $($_.Exception.Message)"
}
finally {
    $witnessKey.Dispose()
    Remove-Variable witnessKey, witnessAccount
}
```

Use the Key value, not a connection string. Enter it only at the local secure prompt. The verified recovery reused an existing key; key regeneration was not required. This changes quorum configuration and is not a routine startup command. See [Microsoft's witness configuration guidance](https://learn.microsoft.com/en-us/windows-server/failover-clustering/deploy-quorum-witness).

#### E. Verify and record closure (read-only)

Repeat the node, quorum, witness, group and CSV checks from step A. Required recovery results are both nodes Up, QuorumResource Cloud Witness, witness and Cluster Group Online, and all CSVs Online. Check virtual-disk health and outstanding storage jobs separately if storage symptoms remain.

Record the observed outcome and unresolved causes. Keep permanent governance follow-up open until it is resolved; a healthy witness today does not prove settings will persist through future evaluations or restarts. Resume the pending accreditation activity after readiness is verified.

### 21.2 Monitor Azure Data and Track Platform Updates

The [monitoring and update evidence](11_Azure_Monitoring_and_Platform_Update_Validation.md) records the executed sequence and current status. Basic monitoring, readiness, preparation, installation to 12.2608.1003.9 and the agreed post-update checks are verified. This section is a reusable procedure, not an instruction to start another run.

1. Verify the telemetry extension on both Arc nodes, then inspect actual cluster metrics under Overview > Monitoring. Extension provisioning alone is not proof of data delivery. Separate missing metric values from zero traffic.
2. For Session expired, reauthenticate the portal session before changing cluster configuration. For heartbeat alerts, inspect timestamps and current connection evidence; User response New is not an independent health result.
3. Inspect Updates > Update readiness and individual check timestamps. Compare old critical results with current node, CSV and virtual-disk state. A blank results list is not a PASS.
4. When appropriate, use Check again once and allow the system check to finish. Read HealthState and HealthCheckDate with Get-SolutionUpdateEnvironment. InProgress with a placeholder date is not success or a clock fault.
5. Review the target version, components and reboot uncertainty. In this lab, preparation used the portal Prepare workflow; it downloaded/validated content and performed health checks before installation.
6. With readiness passed, review the single-system update and confirm installation only within an approved maintenance window. Keep the outer host running and avoid competing cluster changes or manual node restarts.
7. Use the following read-only commands, or the active Updates > History run, to inspect progress:

```powershell
Get-Date -Format o

Get-SolutionUpdate |
    Where-Object { $_.Version -eq '12.2608.1003.9' } |
    Format-List Version, State, UpdateStateProperties, HealthState

Get-SolutionUpdateEnvironment |
    Format-List CurrentVersion, HealthState, HealthCheckDate, State
```

The version filter is specific to this recorded run. Prepared is not Installed. AppliedSuccessfully at environment level must be read together with CurrentVersion and the selected update's terminal result. Unknown on unstarted steps is not by itself failure; inspect actual step timing and errors.

After installation completes, verify the target version, system health, node and storage state, Cloud Witness/quorum, Azure connectivity and workload access. The read-only checks in section 21.1 are reusable; its repair commands are conditional and must not be run simply for validation. Record final evidence before changing the platform lifecycle status to PASS.

References: [portal update workflow](https://learn.microsoft.com/en-us/azure/azure-local/update/azure-update-manager-23h2), [health-check troubleshooting](https://learn.microsoft.com/en-us/azure/azure-local/update/update-troubleshooting-23h2), [update phases](https://learn.microsoft.com/en-us/azure/azure-local/update/update-phases-23h2).

### 21.3 Post-update validation context and guest-access recovery

The completed checkpoint is recorded in [docs/11](11_Azure_Monitoring_and_Platform_Update_Validation.md). Keep these execution contexts distinct:

| Task | Correct location |
| --- | --- |
| Get-SolutionUpdateEnvironment, cluster/storage/quorum checks | PowerShell on AzLHOST1 or AzLHOST2; confirm hostname first |
| Azure resource and Connected Machine Run Command operations | Azure Cloud Shell PowerShell in the correct subscription |
| Guest console login | Windows Admin Center > cluster > Virtual machines > workload VM > Connect |

Running cluster cmdlets on LocalBox-Client returned CommandNotFound; this was a wrong execution context, not an update failure. Opening a VM's Azure Overview resource is not the same as signing into its Windows guest.

The workload VM's Arc agent was Enabled (connected). A forgotten local guest password was recovered through Azure Arc Run Command: read-only user and administrator-group checks identified the intended account, then a targeted Set-LocalUser operation passed the new value through ProtectedParameter. The operation returned Succeeded and exit code 0. Windows Admin Center VMConnect subsequently displayed the logged-in guest desktop.

For a future recovery, confirm the exact VM and account, authorized administrative access, and any risk to encrypted files or stored credentials. Prompt securely; never hardcode, print, commit or request the password in chat. Do not run the reset again merely to reproduce evidence. The Run command portal blade in this session provided CLI/PowerShell instructions rather than an inline script editor. No VM rebuild, host-account reset or NLA disablement was required.

References: [Azure Arc Run Command and protected inputs](https://learn.microsoft.com/en-us/azure/azure-arc/servers/run-command), [WAC VMConnect](https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines#manage-a-virtual-machine-through-the-hyper-v-host-vmconnect).

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

Status synchronized on **3 September 2026** with the [VM lifecycle evidence](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) and [monitoring and update evidence](11_Azure_Monitoring_and_Platform_Update_Validation.md). PASS refers to recorded operator evidence, not a new live test by this documentation update. Platform installation to 12.2608.1003.9 and the agreed post-update validation are PASS. Fresh node, CSV, virtual-disk, witness/quorum, guest-login and basic monitoring evidence is recorded in docs/11. This is a point-in-time lab result, not production certification.

The [recovery checkpoint](10_Lab_Recovery_Checkpoint_and_Follow_Up.md) additionally records both nodes Up, Cloud Witness and Cluster Group Online, and all three CSVs Online. Permanent governance resolution remains pending.

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
Workload VM creation                 PASS
Workload NIC/IP validation           PASS
Guest management / Arc validation    PASS
Workload VM stop/start/restart        PASS
Azure monitoring and management      PASS (basic scope; see docs/11)
Azure Local platform update/lifecycle PASS (12.2608.1003.9; see docs/11)
```

Next: review the synchronized evidence and Markdown presentation drafts, then finish accreditation deliverables. No repeat update or VM lifecycle test is needed solely for documentation. Governance expiry, guest Windows activation, network-metric coverage and local repository synchronization remain separately tracked follow-ups.

