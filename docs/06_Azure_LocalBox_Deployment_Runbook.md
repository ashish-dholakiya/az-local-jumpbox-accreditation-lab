# Azure LocalBox Deployment Runbook

## Purpose

This runbook provides a clean, reproducible, step-by-step process for deploying Microsoft Azure Arc Jumpstart LocalBox for an Azure Local accreditation or proof-of-concept lab.

It is intentionally written as an operational guide rather than a project-history document. It records the validated deployment approach used in this lab up to the current verified checkpoint.

The runbook is scoped to:

- Azure region: `australiaeast`
- Azure Local instance location: `australiaeast`
- LocalBox host VM size: `Standard_E32s_v5`
- Official Microsoft source: Azure Arc Jumpstart LocalBox
- Pinned Microsoft source commit: `3f433866757688d926ae6707e9c0041d8e640b82`

Sensitive values such as subscription IDs, tenant IDs, service-principal object IDs, passwords, tokens, and credentials must never be stored in this document or committed to GitHub.

---

## 1. Understand the LocalBox Lab Architecture

LocalBox does not deploy two separate Azure IaaS VMs to represent the Azure Local nodes.

The Microsoft Bicep template deploys one outer Azure VM named `LocalBox-Client`. That VM is configured as a Hyper-V virtualization host. The LocalBox automation then creates the Azure Local lab as nested virtual machines inside that host.

The resulting lab topology is:

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

### Why LocalBox uses this design

LocalBox is a lab and sandbox implementation. The single Azure VM supplies the compute, storage, networking, and nested-virtualization capability required to emulate the Azure Local environment. The Azure Local nodes are then created as nested Hyper-V VMs.

This allows engineers to exercise Azure Local deployment, Azure Arc integration, cluster validation, lifecycle operations, and management workflows without requiring separate physical Azure Local servers for the lab.

This must not be confused with a production Azure Local architecture. Production Azure Local normally runs on supported and validated physical Azure Local hardware.

### Microsoft source references

Outer Azure VM definition:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

Nested VM configuration:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

Nested Azure Local node creation:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

---

## 2. Prepare the Workstation

The validated workstation baseline is:

```text
PowerShell 7.6.5
Git 2.55.0.windows.5
Azure CLI 2.89.1
Bicep CLI 0.46.1
Az.Accounts 5.5.2
Az.Resources 10.2.0
```

Exact future versions can differ, but PowerShell 7, Git, Azure CLI, Bicep, and current Azure PowerShell modules should be available before deployment.

### Install PowerShell 7

From Windows PowerShell or an elevated-compatible terminal:

```powershell
winget install --id Microsoft.PowerShell --source winget
```

Open a new PowerShell 7 session:

```powershell
pwsh
$PSVersionTable.PSVersion
```

### Verify Git and Azure CLI

```powershell
git --version
az version
```

### Install Bicep

```powershell
az bicep install
az bicep version
```

### Install or update Az.Resources

```powershell
Install-Module Az.Resources `
    -Scope CurrentUser `
    -Repository PSGallery `
    -Force

Get-Module -ListAvailable Az.Resources |
    Sort-Object Version -Descending |
    Select-Object -First 1 Name,Version,Path
```

Verify Az.Accounts as well:

```powershell
Get-Module -ListAvailable Az.Accounts |
    Sort-Object Version -Descending |
    Select-Object -First 1 Name,Version,Path
```

---

## 3. Clone the Official Microsoft LocalBox Source

The lab uses the official Microsoft Azure Arc Jumpstart repository.

```powershell
Set-Location "C:\Projects"

git clone https://github.com/microsoft/azure_arc.git
```

Pin the exact validated LocalBox source commit:

```powershell
Set-Location "C:\Projects\azure_arc"

git checkout 3f433866757688d926ae6707e9c0041d8e640b82

git status
git rev-parse HEAD
```

Expected commit:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

Detached HEAD is intentional because the lab uses the exact tested Microsoft source revision.

The LocalBox Bicep directory is:

```text
C:\Projects\azure_arc\azure_jumpstart_localbox\bicep
```

Do not copy only `main.bicep` to another folder. LocalBox is a multi-module Bicep project and depends on its relative folder structure, including `host`, `mgmt`, and `network` modules.

---

## 4. Authenticate to the Correct Azure Subscription

Authenticate with Azure CLI:

```powershell
az login
```

Select the intended accreditation or lab subscription:

```powershell
az account set --subscription "<SUBSCRIPTION-NAME>"
```

Verify without printing sensitive identifiers:

```powershell
az account show `
    --query "{Subscription:name,State:state,IsDefault:isDefault}" `
    --output table
```

Authenticate Azure PowerShell:

```powershell
Disable-AzContextAutosave -Scope Process | Out-Null
Connect-AzAccount
```

Verify the selected context:

```powershell
$ctx = Get-AzContext

Write-Host "Account present:" ($null -ne $ctx.Account)
Write-Host "Subscription:" $ctx.Subscription.Name
Write-Host "Tenant present:" ($null -ne $ctx.Tenant.Id)
Write-Host "Environment:" $ctx.Environment.Name
```

Do not save tenant IDs, subscription IDs, passwords, or service-principal object IDs in public evidence.

---

## 5. Verify Required Azure Resource Providers Before Deployment

This step is important.

LocalBox contains automation that attempts to register missing providers. However, Azure provider registration is asynchronous and may not complete within the LocalBox polling window. A strict Azure Local validation can therefore fail later even though LocalBox already requested registration.

Verify the mandatory providers before cluster validation:

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

Every required provider should eventually show:

```text
Registered
```

If a provider is not registered, request registration:

```powershell
az provider register --namespace Microsoft.HybridContainerService
az provider register --namespace Microsoft.EdgeMarketplace
```

Then monitor until registration is complete:

```powershell
az provider show `
    --namespace Microsoft.HybridContainerService `
    --query "registrationState" `
    --output tsv

az provider show `
    --namespace Microsoft.EdgeMarketplace `
    --query "registrationState" `
    --output tsv
```

Do not proceed with Azure Local cluster validation until required providers are `Registered`.

---

## 6. Use the Validated Australia East LocalBox Configuration

Validated non-secret deployment values:

| Parameter | Value |
| --- | --- |
| Azure resource location | `australiaeast` |
| Azure Local instance location | `australiaeast` |
| LocalBox host VM size | `Standard_E32s_v5` |
| LocalBox host VM | `LocalBox-Client` |
| Deploy Bastion | `false` |
| Auto-deploy cluster resource | `true` |
| Auto-upgrade cluster resource | `false` |
| Spot pricing | `false` |
| VM auto-logon | `true` |
| Microsoft source reference | exact pinned commit |

The deployment uses the exact Microsoft commit as the `githubBranch` value:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

This prevents the runtime automation from silently moving to a newer `main` branch state during the lab.

The Windows administrator password must be supplied only at runtime and must not be printed, stored in files, or committed to source control.

---

## 7. Create the Australia East Resource Group

Example resource group:

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

Required identity-related inputs include:

- Azure tenant ID
- Microsoft.AzureStackHCI resource-provider service-principal object ID
- Windows administrator password

Retrieve identity values from the authenticated Azure session, but do not print or commit them.

Example tenant retrieval:

```powershell
$tenantId = az account show --query tenantId -o tsv
```

Example Azure Local resource-provider object lookup:

```powershell
$spnProviderId = az ad sp list `
    --filter "appId eq '1412d89f-b8a8-4111-b4fd-e82905cbd85d'" `
    --query "[0].id" `
    -o tsv
```

Capture the LocalBox administrator password securely:

```powershell
$localBoxPassword = Read-Host "Enter LocalBox Windows administrator password" -AsSecureString
```

For the Azure PowerShell deployment call used in this lab, the secure value was converted only transiently in process memory when required by the deployment cmdlet. The plaintext value was never printed, written to disk, or committed.

---

## 9. Validate the LocalBox ARM Request Before Deployment

Use the official Microsoft Bicep template and validated parameters to perform ARM validation before creating the billable host resources.

A known ARM validation diagnostic can occur:

```text
NestedDeploymentShortCircuited
```

In the pinned LocalBox template, the nested host module consumes outputs from earlier modules, such as the staging storage account and subnet. Those values are not fully resolvable during top-level predeployment validation.

Treat this correctly as:

```text
PASS WITH DOCUMENTED VALIDATION LIMITATION
```

It is not full execution validation of the host deployment.

---

## 10. Deploy the LocalBox Host Environment

Run the official LocalBox Bicep deployment using the validated Australia East configuration and the exact pinned Microsoft runtime source reference.

The successful host-level deployment creates Azure resources including:

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

The outer Azure VM should be running after deployment.

Do not stop or deallocate the VM while LocalBox bootstrap and cluster automation are still running.

---

## 11. Verify Bootstrap and Hyper-V Readiness

The `Bootstrap` Custom Script Extension downloads and configures the LocalBox automation.

Relevant source:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

Bootstrap performs activities including:

- LocalBox folder preparation
- PowerShell module installation
- Hyper-V installation
- LocalBox script download
- scheduled-task creation
- optional VM auto-logon

Hyper-V installation requires a restart before the nested virtualization workflow can continue.

A read-only guest-state check can be executed through Azure VM Run Command:

```powershell
az vm run-command invoke `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "LocalBox-Client" `
    --command-id RunPowerShellScript `
    --scripts "Write-Output '=== LAST BOOT TIME ==='; (Get-CimInstance Win32_OperatingSystem).LastBootUpTime; Write-Output '=== HYPER-V STATE ==='; Get-WindowsFeature Hyper-V | Select-Object Name,InstallState | Format-Table -AutoSize; Write-Output '=== LOCALBOX SCHEDULED TASK ==='; Get-ScheduledTask -TaskName 'LocalBoxLogonScript' -ErrorAction SilentlyContinue | Select-Object TaskName,State | Format-Table -AutoSize; Write-Output '=== RECENT LOCALBOX LOGS ==='; Get-ChildItem 'C:\LocalBox\Logs' -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending | Select-Object -First 15 Name,Length,LastWriteTime | Format-Table -AutoSize" `
    --output json
```

Expected checkpoint:

```text
Hyper-V : Installed
LocalBoxLogonScript : Running
```

Useful logs include:

```text
C:\LocalBox\Logs\Bootstrap.log
C:\LocalBox\Logs\LocalBoxLogonScript.log
C:\LocalBox\Logs\New-LocalBoxCluster.log
```

---

## 12. Monitor Nested Azure Local Environment Creation

`New-LocalBoxCluster.ps1` creates and configures the nested lab.

Source:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

The script performs eleven high-level build stages, including:

1. Download LocalBox VHDs.
2. Prepare the Azure VM virtualization host.
3. Create the management VM.
4. Create Azure Local node VMs.
5. Start nested VMs.
6. Configure host networking and storage.
7. Build router VM.
8. Build domain controller VM.
9. Prepare Azure Local cluster cloud deployment.
10. Validate Azure Local cluster deployment.
11. Run Azure Local cluster deployment.

At the nested-node bootstrap stage, Arc registration must complete successfully before cluster deployment can proceed.

---

## 13. Understand the Mandatory Provider Validation Behavior

Before Step 10, the pinned Microsoft script checks these mandatory providers:

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

If a provider is missing, LocalBox calls `Register-AzResourceProvider` and polls for up to approximately five minutes.

If provider registration has not propagated by the end of that polling period, LocalBox continues and allows the Azure Local Environment Validator to perform the strict readiness check.

This matters because provider registration can take longer than the LocalBox polling window.

---

## 14. Diagnose Step 10 Azure Local Validation

The Azure Local cluster validation deployment is named:

```text
localcluster-validate
```

Check its state:

```powershell
az deployment group show `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "localcluster-validate" `
    --query "{Name:name,State:properties.provisioningState,Timestamp:properties.timestamp,Error:properties.error}" `
    --output json
```

If validation fails, inspect failed ARM deployment operations:

```powershell
az deployment operation group list `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "localcluster-validate" `
    --query "[?properties.provisioningState=='Failed'].{OperationId:operationId,Resource:properties.targetResource.resourceName,Type:properties.targetResource.resourceType,State:properties.provisioningState,StatusMessage:properties.statusMessage}" `
    --output json
```

In the validated lab, the earlier Azure Local checks succeeded through:

```text
Azure Local Remote Management
Azure Local Connectivity
Azure Local External Active Directory
Azure Local SBE Health
Azure Local Hardware
Azure Local Network Configuration
Azure Local Network Infra Connection
Azure Local Network Storage Connection
Azure Local Observability
Azure Local Software
Azure Local MOC Stack
```

The blocking check was:

```text
Azure Local Arc Integration
```

The root cause was that `Microsoft.HybridContainerService` had not completed resource-provider registration.

After manually verifying and completing registration for:

```text
Microsoft.HybridContainerService
Microsoft.EdgeMarketplace
```

the Azure Local validation was retried.

---

## 15. Retry Only the Failed Azure Local Validation

Do not redeploy the entire LocalBox environment when the outer host, nested environment, and earlier validation stages are already healthy.

Use the already-generated LocalBox ARM files from inside `LocalBox-Client`:

```text
C:\LocalBox\azlocal.json
C:\LocalBox\azlocal.parameters.json
```

Retry only the validation deployment:

```powershell
az vm run-command invoke `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "LocalBox-Client" `
    --command-id RunPowerShellScript `
    --scripts "Import-Module Az.Resources; Connect-AzAccount -Identity | Out-Null; Write-Output '=== RETRYING AZURE LOCAL VALIDATION ==='; New-AzResourceGroupDeployment -Name 'localcluster-validate' -ResourceGroupName 'rg-azlocal-localbox-accreditation-aue' -TemplateFile 'C:\LocalBox\azlocal.json' -TemplateParameterFile 'C:\LocalBox\azlocal.parameters.json' -ErrorAction Stop | Select-Object DeploymentName,ProvisioningState,Timestamp | Format-List" `
    --output json
```

### Verified result

```text
DeploymentName    : localcluster-validate
ProvisioningState : Succeeded
```

This is the current fully verified checkpoint of the runbook.

---

## 16. Next Step: Run the Actual Azure Local Cluster Deployment

The next LocalBox stage is Step 11/11:

```text
localcluster-deploy
```

The Microsoft LocalBox script starts this deployment only after `localcluster-validate` succeeds.

The deployment can be initiated using the already-generated LocalBox template and parameters:

```powershell
az vm run-command invoke `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "LocalBox-Client" `
    --command-id RunPowerShellScript `
    --scripts "Import-Module Az.Resources; Connect-AzAccount -Identity | Out-Null; Write-Output '=== STARTING AZURE LOCAL CLUSTER DEPLOYMENT ==='; New-AzResourceGroupDeployment -Name 'localcluster-deploy' -ResourceGroupName 'rg-azlocal-localbox-accreditation-aue' -TemplateFile 'C:\LocalBox\azlocal.json' -DeploymentMode 'Deploy' -TemplateParameterFile 'C:\LocalBox\azlocal.parameters.json' -ErrorAction Stop | Select-Object DeploymentName,ProvisioningState,Timestamp | Format-List" `
    --output json
```

### Current status at the time of this runbook update

```text
localcluster-validate : Succeeded
localcluster-deploy   : Final result not yet recorded
```

Do not record Step 11 as successful until the actual deployment result has been captured and verified.

---

## 17. Final Validation After Step 11

After `localcluster-deploy` succeeds, verify the Azure Local cluster separately from the ARM resource creation state.

Recommended checks include:

```powershell
az resource show `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --resource-type "Microsoft.AzureStackHCI/clusters" `
    --name "localboxcluster" `
    --query "{Name:name,ProvisioningState:properties.provisioningState,ConnectivityStatus:properties.connectivityStatus,Status:properties.status}" `
    --output json
```

The desired end state should include operational connectivity, not merely an ARM `ProvisioningState` of `Succeeded`.

The LocalBox test workflow also waits for Azure Local connectivity and runs Pester validation. The pinned test implementation uses a default connectivity timeout of 60 minutes and checks every 30 seconds.

Microsoft test source:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/tests/Invoke-Test.ps1

Additional final checks should cover the Activity 3 outcomes:

- Azure Local cluster connectivity
- Azure Arc connectivity
- Azure Local VM creation and lifecycle
- logical workload networking
- Azure monitoring and management
- Azure Local update and lifecycle capabilities

These sections should be updated only after they are actually executed and verified.

---

## 18. Troubleshooting Principles

Use these rules when troubleshooting LocalBox:

1. Do not assume `ProvisioningState = Succeeded` means the Azure Local cluster is operational.
2. Check `ConnectivityStatus`, cluster `Status`, and deployment-settings validation separately.
3. Inspect ARM deployment operations before changing configuration.
4. Verify resource-provider registration before rerunning validation.
5. Do not restart, stop, or deallocate `LocalBox-Client` while LocalBox automation is running.
6. Do not rerun the full LocalBox deployment when a narrower validation retry is sufficient.
7. Preserve Microsoft source as-is. Do not modify the pinned LocalBox repository unless a change is specifically required and justified.
8. Keep secrets out of scripts, screenshots, GitHub, and public evidence.
9. Treat the Microsoft source code as the implementation authority for LocalBox-specific behavior.
10. Record only verified results as completed.

---

## 19. Quick Reference URLs

Microsoft Azure Arc Jumpstart repository:

https://github.com/microsoft/azure_arc

Pinned LocalBox Bicep host definition:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

Pinned LocalBox cluster automation:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

Pinned LocalBox configuration:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

Pinned LocalBox validation tests:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/tests/Invoke-Test.ps1

---

## Current Verified Checkpoint

```text
Workstation tooling                 PASS
Official Microsoft source cloned    PASS
Microsoft source pinned             PASS
Azure authentication                PASS
Australia East configuration        PASS
Mandatory provider readiness        PASS
LocalBox host deployment            PASS
Hyper-V readiness                   PASS
Nested LocalBox automation          PASS
Arc registration                    PASS
Azure Local validation retry        PASS
localcluster-validate               Succeeded
localcluster-deploy                 Final result pending
```

This runbook should be extended only as later deployment, connectivity, lifecycle, and accreditation steps are actually completed and verified.
