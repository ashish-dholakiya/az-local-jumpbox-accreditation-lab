# Azure LocalBox Deployment Runbook

## Purpose

This runbook provides a clean, reproducible, step-by-step process for deploying Microsoft Azure Arc Jumpstart LocalBox for an Azure Local accreditation or proof-of-concept lab.

It is written for an engineer who wants to reproduce the validated lab without maintaining a separate project repository.

Validated baseline:

```text
Azure region                  australiaeast
Azure Local instance region   australiaeast
LocalBox host VM size         Standard_E32s_v5
LocalBox host VM              LocalBox-Client
Azure Local cluster           localboxcluster
Microsoft source commit       3f433866757688d926ae6707e9c0041d8e640b82
```

Never store subscription IDs, tenant IDs, service-principal object IDs, passwords, tokens, or credentials in public documentation.

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

### Why this design is used

LocalBox is a lab and sandbox implementation. The outer Azure VM supplies the compute, storage, networking, and nested virtualization capacity required to emulate Azure Local. This allows engineers to perform Azure Local deployment, Azure Arc integration, validation, management, and lifecycle exercises without requiring separate physical Azure Local servers.

This is not the production Azure Local architecture. Production Azure Local runs on supported and validated physical hardware.

Microsoft references:

Outer Azure VM definition:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

Nested VM configuration:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

Nested Azure Local node creation:

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

Install PowerShell 7 if required:

```powershell
winget install --id Microsoft.PowerShell --source winget
```

Open PowerShell 7 and verify:

```powershell
pwsh
$PSVersionTable.PSVersion
```

Verify Git and Azure CLI:

```powershell
git --version
az version
```

Install Bicep:

```powershell
az bicep install
az bicep version
```

Install or update `Az.Resources`:

```powershell
Install-Module Az.Resources `
    -Scope CurrentUser `
    -Repository PSGallery `
    -Force

Get-Module -ListAvailable Az.Resources |
    Sort-Object Version -Descending |
    Select-Object -First 1 Name,Version,Path
```

---

## 3. Clone the Official Microsoft LocalBox Source

```powershell
Set-Location "C:\Projects"

git clone https://github.com/microsoft/azure_arc.git

Set-Location "C:\Projects\azure_arc"

git checkout 3f433866757688d926ae6707e9c0041d8e640b82

git status
git rev-parse HEAD
```

Expected commit:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

The LocalBox Bicep directory is:

```text
C:\Projects\azure_arc\azure_jumpstart_localbox\bicep
```

Do not copy only `main.bicep` to another directory. LocalBox is a multi-module Bicep project and depends on the relative `host`, `mgmt`, `network`, and supporting folder structure.

---

## 4. Authenticate to Azure

Azure CLI:

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

Azure PowerShell:

```powershell
Disable-AzContextAutosave -Scope Process | Out-Null
Connect-AzAccount
```

Verify:

```powershell
$ctx = Get-AzContext
Write-Host "Account present:" ($null -ne $ctx.Account)
Write-Host "Subscription:" $ctx.Subscription.Name
Write-Host "Tenant present:" ($null -ne $ctx.Tenant.Id)
Write-Host "Environment:" $ctx.Environment.Name
```

---

## 5. Verify Mandatory Resource Providers Before Deployment

Do this before Azure Local validation.

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

Every mandatory provider should be:

```text
Registered
```

If required, request registration:

```powershell
az provider register --namespace Microsoft.HybridContainerService
az provider register --namespace Microsoft.EdgeMarketplace
```

Then continue checking until both show `Registered`.

### Why this check matters

LocalBox can request missing provider registration automatically, but Azure provider registration is asynchronous. The LocalBox polling window may finish before registration has propagated. The Azure Local Environment Validator can then correctly fail Arc Integration readiness.

Pre-registering and verifying these providers avoids that failure.

---

## 6. Use the Australia East Configuration

Validated non-secret values:

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

Use the exact commit as the LocalBox runtime `githubBranch` value:

```text
3f433866757688d926ae6707e9c0041d8e640b82
```

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

Retrieve the tenant ID and Microsoft.AzureStackHCI resource-provider object ID from the authenticated Azure context without printing or storing them in public evidence.

Example:

```powershell
$tenantId = az account show --query tenantId -o tsv

$spnProviderId = az ad sp list `
    --filter "appId eq '1412d89f-b8a8-4111-b4fd-e82905cbd85d'" `
    --query "[0].id" `
    -o tsv
```

Capture the LocalBox administrator password securely:

```powershell
$localBoxPassword = Read-Host "Enter LocalBox Windows administrator password" -AsSecureString
```

Do not print or save the password.

---

## 9. Validate the LocalBox ARM Request

Use the official `main.bicep` and the validated parameter set with `Test-AzResourceGroupDeployment` before creating billable resources.

A diagnostic named:

```text
NestedDeploymentShortCircuited
```

can appear because the nested host module depends on outputs from earlier modules, such as the staging storage account and subnet.

Treat a successful top-level validation with this documented nested-evaluation limitation as:

```text
PASS WITH DOCUMENTED VALIDATION LIMITATION
```

It is not full execution validation of the nested host deployment.

---

## 10. Deploy the LocalBox Host

Run the official Microsoft LocalBox Bicep deployment with:

```text
location                    australiaeast
azureLocalInstanceLocation  australiaeast
vmSize                      Standard_E32s_v5
githubBranch                exact pinned commit
```

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

## 11. Verify Bootstrap and Hyper-V

Read-only guest check:

```powershell
az vm run-command invoke `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "LocalBox-Client" `
    --command-id RunPowerShellScript `
    --scripts "Write-Output '=== LAST BOOT TIME ==='; (Get-CimInstance Win32_OperatingSystem).LastBootUpTime; Write-Output '=== HYPER-V STATE ==='; Get-WindowsFeature Hyper-V | Select-Object Name,InstallState | Format-Table -AutoSize; Write-Output '=== LOCALBOX SCHEDULED TASK ==='; Get-ScheduledTask -TaskName 'LocalBoxLogonScript' -ErrorAction SilentlyContinue | Select-Object TaskName,State | Format-Table -AutoSize; Write-Output '=== RECENT LOCALBOX LOGS ==='; Get-ChildItem 'C:\LocalBox\Logs' -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending | Select-Object -First 15 Name,Length,LastWriteTime | Format-Table -AutoSize" `
    --output json
```

Expected during build:

```text
Hyper-V : Installed
LocalBoxLogonScript : Running
```

Useful logs:

```text
C:\LocalBox\Logs\Bootstrap.log
C:\LocalBox\Logs\LocalBoxLogonScript.log
C:\LocalBox\Logs\New-LocalBoxCluster.log
```

---

## 12. Monitor the Nested LocalBox Build

Microsoft automation:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/New-LocalBoxCluster.ps1

The workflow performs eleven high-level stages:

1. Download VHDs.
2. Prepare the virtualization host.
3. Create the management VM.
4. Create Azure Local node VMs.
5. Start nested VMs.
6. Configure networking and storage.
7. Build router VM.
8. Build domain controller VM.
9. Prepare Azure Local cloud deployment.
10. Validate Azure Local cluster deployment.
11. Deploy Azure Local cluster.

---

## 13. Validate the Azure Local Cluster

The validation deployment is:

```text
localcluster-validate
```

Check state:

```powershell
az deployment group show `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "localcluster-validate" `
    --query "{Name:name,State:properties.provisioningState,Timestamp:properties.timestamp,Error:properties.error}" `
    --output json
```

If validation fails, inspect operations:

```powershell
az deployment operation group list `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "localcluster-validate" `
    --query "[?properties.provisioningState=='Failed'].{Resource:properties.targetResource.resourceName,Type:properties.targetResource.resourceType,State:properties.provisioningState,StatusMessage:properties.statusMessage}" `
    --output json
```

If provider readiness was the only blocker and the LocalBox environment is otherwise healthy, correct the provider registration and retry only validation, not the complete LocalBox deployment.

Validated retry command:

```powershell
az vm run-command invoke `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "LocalBox-Client" `
    --command-id RunPowerShellScript `
    --scripts "Import-Module Az.Resources; Connect-AzAccount -Identity | Out-Null; New-AzResourceGroupDeployment -Name 'localcluster-validate' -ResourceGroupName 'rg-azlocal-localbox-accreditation-aue' -TemplateFile 'C:\LocalBox\azlocal.json' -TemplateParameterFile 'C:\LocalBox\azlocal.parameters.json' -ErrorAction Stop | Select-Object DeploymentName,ProvisioningState,Timestamp | Format-List" `
    --output json
```

Required result:

```text
localcluster-validate : Succeeded
```

---

## 14. Deploy the Azure Local Cluster

After validation succeeds, deploy the cluster using the generated LocalBox ARM files:

```powershell
az vm run-command invoke `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "LocalBox-Client" `
    --command-id RunPowerShellScript `
    --scripts "Import-Module Az.Resources; Connect-AzAccount -Identity | Out-Null; New-AzResourceGroupDeployment -Name 'localcluster-deploy' -ResourceGroupName 'rg-azlocal-localbox-accreditation-aue' -TemplateFile 'C:\LocalBox\azlocal.json' -DeploymentMode 'Deploy' -TemplateParameterFile 'C:\LocalBox\azlocal.parameters.json' -ErrorAction Stop | Select-Object DeploymentName,ProvisioningState,Timestamp | Format-List" `
    --output json
```

### Important long-running deployment behavior

The Azure Local cluster deployment can run for hours. In the validated lab, it completed in approximately:

```text
2 hours 19 minutes
```

The synchronous Azure VM Run Command wrapper timed out before the deployment completed. This did not mean the Azure Local deployment failed.

Once the ARM deployment has been started, use Azure Resource Manager as the authoritative monitor:

```powershell
az deployment group show `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "localcluster-deploy" `
    --query "{State:properties.provisioningState,Duration:properties.duration,Error:properties.error}" `
    --output json
```

Inspect non-succeeded child operations if needed:

```powershell
az deployment operation group list `
    --resource-group "rg-azlocal-localbox-accreditation-aue" `
    --name "localcluster-deploy" `
    --query "[?properties.provisioningState!='Succeeded'].{Resource:properties.targetResource.resourceName,Type:properties.targetResource.resourceType,State:properties.provisioningState,StatusMessage:properties.statusMessage}" `
    --output json
```

Do not rerun the deployment simply because the initiating Run Command timed out.

Required final ARM result:

```text
State : Succeeded
Error : null
```

---

## 15. Inspect Detailed Azure Local Deployment Status

For deeper deployment progress, query the nested Azure Local deployment settings directly through ARM REST.

First obtain the current subscription ID in process memory:

```powershell
$subscriptionId = az account show --query id --output tsv
```

Use the stable API version validated for the resource type:

```powershell
$uri = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/rg-azlocal-localbox-accreditation-aue/providers/Microsoft.AzureStackHCI/clusters/localboxcluster/deploymentSettings/default?api-version=2026-04-30"

az rest `
    --method get `
    --uri $uri `
    --query "properties" `
    --output json
```

Successful deployment should report values such as:

```text
provisioningState       : Succeeded
deploymentMode          : Deploy
deploymentStatus.status : Success
validationStatus.status : Success
```

This detailed status also shows the individual deployment and validation steps.

---

## 16. Verify Final Azure Local Connectivity

After deployment succeeds:

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

Do not treat ARM provisioning success alone as operational completion. `ConnectivityStatus = Connected` is a separate completion gate.

---

## 17. Verify the Two Azure Local Nodes

Detailed deployment settings should contain Arc node resource IDs for:

```text
AzLHOST1
AzLHOST2
```

These are the two Azure Local nodes projected through Azure Arc / Hybrid Compute. They are nested inside `LocalBox-Client`, not separate top-level Azure IaaS VMs.

---

## 18. Microsoft Validation Tests

Pinned test source:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/tests/Invoke-Test.ps1

The Microsoft test workflow waits for deployment completion and Azure Local connectivity before running Pester validation.

The connectivity check uses a default 60-minute timeout and checks every 30 seconds.

---

## 19. Troubleshooting Rules

1. Verify provider registration before cluster validation.
2. Do not assume `ProvisioningState = Succeeded` means Azure Local is operationally connected.
3. Check `ConnectivityStatus` separately.
4. Inspect ARM deployment operations before changing configuration.
5. A VM Run Command timeout is not automatically an Azure Local deployment failure.
6. Monitor long-running deployment state through Azure Resource Manager.
7. Do not restart, stop, or deallocate `LocalBox-Client` during active LocalBox automation.
8. Do not rerun the complete LocalBox deployment when a narrower validation retry is sufficient.
9. Keep Microsoft source pinned and unmodified unless a justified change is explicitly required.
10. Keep secrets and sensitive IDs out of screenshots, scripts, repositories, and evidence.

---

## 20. Quick Reference URLs

Microsoft Azure Arc Jumpstart repository:

https://github.com/microsoft/azure_arc

Pinned LocalBox host definition:

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
```

Remaining accreditation Activity 3 operational exercises should now continue one at a time:

1. Azure Local VM creation and lifecycle.
2. Logical workload networking.
3. Azure monitoring and management.
4. Azure Local update and lifecycle validation.
