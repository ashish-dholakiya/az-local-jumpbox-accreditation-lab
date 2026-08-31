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

The Windows administrator password is not stored in this public repository. It is supplied only at runtime.

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

## 8. Verify Local Deployment Tooling Readiness

### What was done

The local deployment tooling required for the official LocalBox Bicep workflow was verified.

Verified components:

```text
Azure CLI 2.89.1
Bicep CLI 0.46.1
Az.Resources 10.1.0
Az.Accounts 5.5.2
```

The Azure CLI-managed Bicep executable was added to the current PowerShell process PATH so Azure PowerShell could invoke Bicep during template validation.

Azure PowerShell authentication was also completed and the active context was verified against the correct accreditation subscription. Sensitive account and tenant identifiers are intentionally not stored in this public repository.

### Status

**PASS.** Local Azure CLI, Bicep, Azure PowerShell deployment cmdlets, and the correct Azure PowerShell subscription context are ready.

### Change impact

Local workstation configuration and authentication context only. No Azure workload resources were created.

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

The deployment configuration therefore uses:

```text
githubBranch = 3f433866757688d926ae6707e9c0041d8e640b82
```

instead of a moving `main` reference.

### Direct Bicep provider namespaces discovered

```text
Microsoft.Authorization
Microsoft.Compute
Microsoft.Network
Microsoft.OperationalInsights
Microsoft.Resources
Microsoft.Storage
```

Initial verification found `Microsoft.Network` and `Microsoft.OperationalInsights` unregistered. Both were then registered and re-verified.

### Final verified provider registration state

```text
Microsoft.Authorization : Registered
Microsoft.Compute : Registered
Microsoft.Network : Registered
Microsoft.OperationalInsights : Registered
Microsoft.Resources : Registered
Microsoft.Storage : Registered
```

### Status

**PASS.** Runtime exact-commit artifact reachability is validated and all directly referenced provider namespaces are registered.

### Change impact

**Azure subscription change.** `Microsoft.Network` and `Microsoft.OperationalInsights` were registered at the subscription level. No LocalBox workload resources were created by provider registration.

---

## 10. Validate Secure Runtime Inputs and Final ARM Deployment Request

### What was done

The LocalBox Windows administrator password was captured through a PowerShell secure prompt and verified as a `System.Security.SecureString` without displaying the secret.

The mandatory tenant and Azure Local resource-provider object ID values were retrieved in the authenticated Azure PowerShell session and validated only as present. Their actual values were not recorded in public evidence.

The complete LocalBox parameter set was then assembled using the locked deployment configuration, including the exact Microsoft commit for `githubBranch`.

Azure PowerShell `Test-AzResourceGroupDeployment` was used to perform an ARM predeployment validation against the dedicated accreditation resource group.

Because the cmdlet could not serialize a `SecureString` inside `TemplateParameterObject`, the password was converted only transiently in PowerShell process memory for the validation request. The temporary plaintext variable/reference was removed immediately afterward. The password was never printed, written to a file, or committed to GitHub.

### Verified result

```text
LocalBox ARM validation: PASS
Temporary plaintext password reference cleared.
```

Azure returned one diagnostic warning:

```text
NestedDeploymentShortCircuited
```

The warning applied to the nested `hostVmDeployment`. Inspection of the pinned Microsoft `main.bicep` confirmed that this host module consumes outputs from earlier modules, including the staging storage account name and network subnet ID. Those values are deployment-time outputs and cannot be fully evaluated during this validation request.

The warning therefore represents an ARM nested-deployment validation limitation, not a detected template syntax error, provider-registration failure, password error, or missing LocalBox input.

### Status

**PASS WITH DOCUMENTED VALIDATION LIMITATION.** The top-level LocalBox ARM deployment request and supplied parameters passed validation. The nested host deployment could not be fully prevalidated because it depends on deployment-time module outputs.

This must not be represented as full execution validation of all host resources. Actual host resource creation remains untested until the billable LocalBox deployment is performed.

### Change impact

**Validation only.** No LocalBox VM, network, storage, Log Analytics, Azure Local cluster, or other billable workload resource was created by this step.

---

## 11. Current Activity 3 Implementation Status

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
| Azure PowerShell deployment tooling | PASS |
| Azure PowerShell subscription context | PASS |
| Runtime source-reference validation | PASS |
| Runtime artifact exact-commit reachability | PASS |
| Direct provider dependency discovery | PASS |
| Microsoft.Network registration | PASS |
| Microsoft.OperationalInsights registration | PASS |
| Secure runtime password input | PASS |
| Final ARM predeployment validation | PASS WITH DOCUMENTED LIMITATION |
| Billable LocalBox resource deployment | NOT STARTED |
| Azure Local cluster deployment / review | NOT STARTED |
| Azure Arc validation | NOT STARTED |
| Azure Local VM lifecycle validation | NOT STARTED |
| Logical workload networking validation | NOT STARTED |
| Azure monitoring and management validation | NOT STARTED |
| Azure Local update / lifecycle validation | NOT STARTED |

## Next Required Step

The predeployment readiness gate is complete. The next implementation action is the billable LocalBox deployment using the validated parameter set and the exact pinned Microsoft runtime source reference.

Before starting that deployment, explicitly confirm the cost impact and deployment window. Once deployment begins, monitor the Azure resource-group deployment and preserve sanitized evidence of the deployment result.