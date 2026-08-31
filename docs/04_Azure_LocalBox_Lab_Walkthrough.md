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

The planned lab configuration continues to use Central India, subject to final parameter validation before resource creation.

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

## 5. Current Activity 3 Implementation Status

| Implementation checkpoint | Status |
| --- | --- |
| Azure subscription readiness | PASS |
| Official Microsoft Jumpstart repository reachable | PASS |
| Exact LocalBox source commit pinned | PASS |
| LocalBox Bicep deployment path identified | PASS |
| Required template parameter structure reviewed | PASS |
| Tenant ID input retrieved | PASS |
| Azure Local RP object ID retrieved | PASS |
| Final deployment parameter values locked | PENDING |
| Billable LocalBox resource deployment | NOT STARTED |
| Azure Local cluster deployment / review | NOT STARTED |
| Azure Arc validation | NOT STARTED |
| Azure Local VM lifecycle validation | NOT STARTED |
| Logical workload networking validation | NOT STARTED |
| Azure monitoring and management validation | NOT STARTED |
| Azure Local update / lifecycle validation | NOT STARTED |

## Next Required Step

Before any billable Azure resource is created, lock and validate the exact LocalBox deployment parameter values. This includes the Azure resource location, Azure Local instance location, VM size, administrator username, secure password handling, Bastion decision, cluster auto-deployment setting and cost-related options.
