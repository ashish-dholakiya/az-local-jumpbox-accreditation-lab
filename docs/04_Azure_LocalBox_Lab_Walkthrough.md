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

The official Microsoft `azure_arc` repository was cloned and checked out at the exact verified commit rather than relying on a moving branch state.

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

**PASS.** Official LocalBox deployment source is pinned locally.

### Change impact

Local filesystem only. No Azure workload resources were created.

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

The two mandatory identity-related LocalBox inputs were retrieved without printing their actual values into the evidence output.

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

### Verified configuration

| Parameter | Locked value | Reason |
| --- | --- | --- |
| Resource group | `rg-azlocal-localbox-accreditation-ci` | Dedicated accreditation lab boundary and easy cleanup. |
| `location` | `centralindia` | Keeps supporting Azure resources in the selected lab region. |
| `azureLocalInstanceLocation` | `centralindia` | Registers the Azure Local instance in the selected region and is allowed by the pinned LocalBox template. |
| `vmSize` | `Standard_E32s_v6` | Supported by the pinned LocalBox template and validated for Central India quota and availability. |
| `windowsAdminUsername` | `arcdemo` | Uses the LocalBox reference username. |
| `deployBastion` | `false` | Avoids an additional service unless a required lab outcome depends on it. |
| `autoDeployClusterResource` | `true` | Supports the required Activity 3 Azure Local cluster deployment / review outcome. |
| `autoUpgradeClusterResource` | `false` | Cluster deployment will be validated before any upgrade workflow is introduced. |
| `enableAzureSpotPricing` | `false` | Avoids Spot eviction risk during an accreditation lab. |
| `vmAutologon` | `true` | Retains the LocalBox lab automation behavior. |
| `governResourceTags` | `false` | Avoids assuming that Microsoft-internal lab governance tags apply to this subscription. |

### Security note

The Windows administrator password is not stored in this public repository. It will be supplied securely at deployment time.

### Status

**PASS.** Final non-secret LocalBox deployment parameter values are locked and verified.

---

## 6. Create and Verify the Dedicated Azure Resource Group

### What was done

A dedicated Azure resource group was created in Central India to contain the accreditation LocalBox deployment and simplify implementation tracking and later cleanup.

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

The accreditation workflow was moved from an ephemeral Cloud Shell session to a persistent local VS Code PowerShell workflow.

The following were verified locally:

- Git `2.55.0`.
- PowerShell `7.6.5`.
- Azure CLI `2.89.1`.
- Accreditation repository cloned to `C:\Projects\az-local-jumpbox-accreditation-lab` on branch `main` with a clean working tree.
- Official Microsoft `azure_arc` repository cloned separately to `C:\Projects\azure_arc`.
- Microsoft source pinned to `3f433866757688d926ae6707e9c0041d8e640b82` with a clean detached HEAD.
- Dedicated Azure CLI profile configured at `C:\AzureCLIProfiles\AzureLocalAccreditation` so the accreditation login context remains isolated from other Azure projects.
- Correct accreditation subscription authenticated and selected in the isolated Azure CLI profile.
- Existing accreditation resource group visible from the local VS Code session with provisioning state `Succeeded`.

A local-only helper script was created at:

```text
C:\AzureCLIProfiles\AzureLocalAccreditation\Resume-AzureLocal.ps1
```

The helper script is not stored in the public accreditation repository.

### Verified resume result

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

**PASS.** The accreditation implementation can be safely resumed after terminal closure or laptop restart without relying on ephemeral PowerShell variables.

---

## 8. Verify Local Bicep CLI Readiness

### What was done

Bicep availability was checked from the isolated Azure CLI profile on the local VS Code workstation. The initial check confirmed that Bicep was not installed. Bicep was then installed through Azure CLI and re-verified.

### Commands

```powershell
az bicep install
az bicep version
```

### Verified result

```text
Bicep CLI version 0.46.1 (545b338e2c)
```

Bicep was installed into the dedicated accreditation Azure CLI profile path rather than the general Azure CLI profile.

### Status

**PASS.** Local Bicep CLI `0.46.1` is available for the official LocalBox Bicep deployment workflow.

### Change impact

**Local workstation configuration only.** No Azure resources were created or modified.

---

## 9. Validate Runtime Source Pinning and Direct Provider Dependencies

### What was done

The pinned LocalBox source was scanned to identify runtime GitHub references and direct Azure resource-provider namespaces used by the Bicep modules.

The scan confirmed that `githubBranch` is used to build a `raw.githubusercontent.com` runtime artifact base URL and defaults to `main` in the official template. A direct HTTP test was then performed using the exact verified Microsoft commit SHA in place of the branch name.

### Verified runtime source result

```text
Pinned runtime artifact reachable: True
HTTP Status: 200
```

This confirms that the exact commit SHA can resolve the tested LocalBox runtime artifact. The deployment configuration can therefore use:

```text
githubBranch = 3f433866757688d926ae6707e9c0041d8e640b82
```

instead of a moving `main` reference, without modifying Microsoft source code.

### Direct Bicep provider namespaces discovered

```text
Microsoft.Authorization
Microsoft.Compute
Microsoft.Network
Microsoft.OperationalInsights
Microsoft.Resources
Microsoft.Storage
```

### Verified provider registration state

```text
Microsoft.Authorization : Registered
Microsoft.Compute : Registered
Microsoft.Network : NotRegistered
Microsoft.OperationalInsights : NotRegistered
Microsoft.Resources : Registered
Microsoft.Storage : Registered
```

### Status

**PARTIAL PASS.** Runtime commit pinning is validated. Four directly referenced providers are already registered. Two directly referenced providers remain unregistered and must be registered before billable LocalBox deployment:

- `Microsoft.Network`
- `Microsoft.OperationalInsights`

### Change impact

**Read-only validation only.** No provider registrations or Azure workload resources were changed by this step.

---

## 10. Current Activity 3 Implementation Status

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
| Local Bicep CLI readiness | PASS |
| Runtime source-reference validation | PASS |
| Runtime artifact exact-commit reachability | PASS |
| Direct provider dependency discovery | PASS |
| Microsoft.Network registration | PENDING |
| Microsoft.OperationalInsights registration | PENDING |
| Billable LocalBox resource deployment | NOT STARTED |
| Azure Local cluster deployment / review | NOT STARTED |
| Azure Arc validation | NOT STARTED |
| Azure Local VM lifecycle validation | NOT STARTED |
| Logical workload networking validation | NOT STARTED |
| Azure monitoring and management validation | NOT STARTED |
| Azure Local update / lifecycle validation | NOT STARTED |

## Next Required Step

Register only the two directly required providers that are currently unregistered, `Microsoft.Network` and `Microsoft.OperationalInsights`, then verify both reach `Registered`. After that, re-synchronize the local accreditation repository and continue to final secure parameter preparation and predeployment validation.