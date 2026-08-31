# Azure LocalBox Lab Walkthrough

## Purpose

This document records the verified implementation steps for Accreditation Activity 3: Azure Local proof of concept using Microsoft Azure Arc Jumpstart LocalBox.

The governing scope and customer scenario are defined in `docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md`. This walkthrough records only actions that have actually been executed and verified. Planned actions are not represented as completed.

## Execution Discipline

The implementation follows this sequence:

**Source of Truth -> Required Task -> Implementation -> Verification -> Evidence -> Customer Explanation**

The Microsoft Jumpstart repository is used as the official LocalBox deployment source. The Microsoft repository remains separate from this accreditation repository. This repository stores the implementation record, decisions and sanitized evidence only.

---

## 1. Verify Official Microsoft Jumpstart Repository Availability

### What was done

The current HEAD of the official Microsoft Azure Arc repository was queried from Azure Cloud Shell.

### Command

```powershell
git ls-remote https://github.com/microsoft/azure_arc.git HEAD
```

### Verified result

```text
3f433866757688d926ae6707e9c0041d8e640b82    HEAD
```

### Status

**PASS.** The official repository was reachable and the exact source commit was identified.

### Change impact

**Read-only.** No Azure resources were created or modified.

---

## 2. Clone and Pin the Official LocalBox Deployment Source

### What was done

The official Microsoft `azure_arc` repository was cloned into Azure Cloud Shell and checked out at the exact verified commit rather than relying on a moving branch state.

### Commands

```powershell
cd ~

git clone https://github.com/microsoft/azure_arc.git
cd ./azure_arc

git checkout 3f433866757688d926ae6707e9c0041d8e640b82

git status
git rev-parse HEAD

Set-Location ./azure_jumpstart_localbox/bicep
Get-ChildItem
```

### Verified result

- Repository clone completed successfully.
- HEAD was pinned to `3f433866757688d926ae6707e9c0041d8e640b82`.
- Git reported a clean working tree.
- Detached HEAD state is intentional because the lab uses the exact verified Microsoft commit.
- The LocalBox Bicep deployment path is `azure_jumpstart_localbox/bicep`.

The pinned Microsoft commit identifies the LocalBox update as:

```text
Update LocalBox to use image version 2608 (#3521)
```

### Status

**PASS.** Official LocalBox deployment source is pinned and reproducible.

### Change impact

Changes only the Cloud Shell working directory by downloading repository files. No Azure workload resources were created.

---

## 3. Confirm LocalBox Deployment Inputs from the Official Template

The pinned template was reviewed before deployment.

Verified relevant deployment parameters include:

- `tenantId`
- `spnProviderId`
- `windowsAdminUsername`
- secure `windowsAdminPassword`
- `location`
- `azureLocalInstanceLocation`
- `vmSize`
- `deployBastion`
- `autoDeployClusterResource`
- `autoUpgradeClusterResource`
- `enableAzureSpotPricing`

The official template supports `Standard_E32s_v6` and includes `centralindia` as an allowed Azure Local instance location.

### Status

**PASS.** Required parameter structure has been identified from the pinned official template.

### Change impact

**Read-only.** Template inspection did not create or modify Azure resources.

---

## 4. Retrieve Mandatory Tenant and Azure Local Resource Provider Inputs

### What was done

The two mandatory identity-related LocalBox inputs were retrieved in Azure Cloud Shell without printing their actual values into the evidence output.

### Commands

```powershell
$tenantId = az account show --query tenantId -o tsv

$spnProviderId = az ad sp list `
    --filter "appId eq '1412d89f-b8a8-4111-b4fd-e82905cbd85d'" `
    --query "[0].id" `
    -o tsv

Write-Host "Tenant ID retrieved:" ($tenantId -ne "")
Write-Host "Azure Local RP Object ID retrieved:" ($spnProviderId -ne "")
```

### Verified result

```text
Tenant ID retrieved: True
Azure Local RP Object ID retrieved: True
```

The actual Tenant ID and Azure Local Resource Provider object ID are intentionally not stored in this public repository.

### Status

**PASS.** Both mandatory identity inputs required by the LocalBox template were successfully retrieved.

### Change impact

**Read-only.** No Azure resources or role assignments were created or modified.

---

## 5. Lock LocalBox Deployment Parameters

### What was done

The deployment configuration was defined and printed in Azure Cloud Shell before any Azure workload resources were created.

### Verified configuration

| Parameter | Locked value | Reason |
| --- | --- | --- |
| Resource group | `rg-azlocal-localbox-accreditation-ci` | Dedicated accreditation lab boundary and easy cleanup. |
| `location` | `centralindia` | Keeps supporting Azure resources in the selected lab region. |
| `azureLocalInstanceLocation` | `centralindia` | Registers the Azure Local instance in the same selected region and is allowed by the pinned LocalBox template. |
| `vmSize` | `Standard_E32s_v6` | Supported by the pinned LocalBox template and previously validated for Central India quota and availability. |
| `windowsAdminUsername` | `arcdemo` | Uses the LocalBox reference username. |
| `deployBastion` | `false` | Avoids an additional service unless the required lab outcome later depends on it. |
| `autoDeployClusterResource` | `true` | Supports the required Activity 3 Azure Local cluster deployment / review outcome. |
| `autoUpgradeClusterResource` | `false` | Cluster deployment will be validated before any optional upgrade workflow is introduced. |
| `enableAzureSpotPricing` | `false` | Avoids Spot eviction risk during an accreditation lab. |
| `vmAutologon` | `true` | Retains the LocalBox lab automation behavior. |
| `governResourceTags` | `false` | The template states its special CostControl and SecurityControl tags are intended for Microsoft-internal Azure lab tenants; they are not assumed for this company subscription. |

### Verification output

The Cloud Shell output confirmed the values above, including:

```text
Resource Group: rg-azlocal-localbox-accreditation-ci
Azure Resource Location: centralindia
Azure Local Instance Location: centralindia
VM Size: Standard_E32s_v6
Windows Admin Username: arcdemo
Deploy Bastion: False
Auto Deploy Cluster: True
Auto Upgrade Cluster: False
Spot Pricing: False
VM Autologon: True
Govern Resource Tags: False
```

### Security note

The Windows administrator password is not stored in this public repository and was intentionally excluded from this parameter-lock evidence. It will be supplied securely at deployment time.

### Status

**PASS.** Final non-secret LocalBox deployment parameter values are locked and verified.

### Change impact

**Local Cloud Shell preparation only.** No Azure workload resources were created or modified.

---

## 6. Create and Verify the Dedicated Azure Resource Group

### What was done

A dedicated Azure resource group was created in Central India to contain the accreditation LocalBox deployment and simplify implementation tracking and later cleanup.

### Command

```powershell
az group create `
    --name $resourceGroupName `
    --location $location `
    --tags Project=AzureLocalAccreditation Environment=Lab Purpose=LocalBox
```

### Verification command

```powershell
az group show `
    --name $resourceGroupName `
    --query "{Name:name,Location:location,ProvisioningState:properties.provisioningState}" `
    -o table
```

### Verified result

```text
Name                                  Location      ProvisioningState
------------------------------------  ------------  -------------------
rg-azlocal-localbox-accreditation-ci  centralindia  Succeeded
```

### Status

**PASS.** The dedicated LocalBox accreditation resource group exists in Central India and its provisioning state is `Succeeded`.

### Change impact

**Azure subscription change.** A resource group and its non-billable container metadata were created. No LocalBox workload resources have been deployed yet.

---

## 7. Establish Persistent Local VS Code Resume Workflow

### What was done

The accreditation workflow was moved from an ephemeral Cloud Shell session to a persistent local VS Code PowerShell workflow on the personal laptop.

The following were verified locally:

- Git `2.55.0`.
- PowerShell `7.6.5`.
- Azure CLI `2.89.1`.
- Accreditation repository cloned to `C:\Projects\az-local-jumpbox-accreditation-lab` on branch `main` with a clean working tree.
- Official Microsoft `azure_arc` repository cloned separately to `C:\Projects\azure_arc`.
- Microsoft source pinned to `3f433866757688d926ae6707e9c0041d8e640b82` with a clean detached HEAD.
- Dedicated Azure CLI profile path configured as `C:\AzureCLIProfiles\AzureLocalAccreditation` so this accreditation login context remains isolated from other Azure projects on the same laptop.
- Correct accreditation subscription was authenticated and selected in the isolated Azure CLI profile.
- Existing accreditation resource group was visible from the local VS Code session with provisioning state `Succeeded`.

A local-only helper script was created at:

```text
C:\AzureCLIProfiles\AzureLocalAccreditation\Resume-AzureLocal.ps1
```

The helper script is not stored in the public accreditation repository. It restores the dedicated Azure CLI profile, verifies both repositories, checks the pinned Microsoft commit, validates the Azure subscription context, verifies the resource group, and returns the working directory to the accreditation repository.

### Verified resume result

The resume script completed successfully and confirmed:

```text
Branch: main
Pinned Commit Match: True
Azure subscription state: Enabled
Azure subscription default context: True
Resource group location: centralindia
Resource group provisioning state: Succeeded
Current Folder: C:\Projects\az-local-jumpbox-accreditation-lab
```

### Status

**PASS.** The accreditation implementation can now be safely resumed after terminal closure, Cloud Shell disconnect, or laptop restart without relying on ephemeral PowerShell variables.

### Change impact

**Local workstation configuration only.** No additional Azure workload resources were created or modified by this resume setup.

---

## 8. Current Activity 3 Implementation Status

| Implementation checkpoint | Status |
| --- | --- |
| Azure subscription readiness | PASS |
| Official Microsoft Jumpstart repository reachable | PASS |
| Exact LocalBox source commit pinned | PASS |
| LocalBox Bicep deployment path identified | PASS |
| Required template parameter structure reviewed | PASS |
| Tenant ID input retrieved | PASS |
| Azure Local RP object ID retrieved | PASS |
| Final deployment parameter values locked | PASS |
| Dedicated LocalBox resource group created | PASS |
| Persistent local VS Code resume workflow | PASS |
| Billable LocalBox resource deployment | NOT STARTED |
| Azure Local cluster deployment / review | NOT STARTED |
| Azure Arc validation | NOT STARTED |
| Azure Local VM lifecycle validation | NOT STARTED |
| Logical workload networking validation | NOT STARTED |
| Azure monitoring and management validation | NOT STARTED |
| Azure Local update / lifecycle validation | NOT STARTED |

## Next Required Step

Prepare the secure Windows administrator password input and validate the final LocalBox deployment command without exposing the password in the public repository. The subsequent LocalBox deployment will create billable Azure resources, so cost and cleanup discipline must remain in effect.