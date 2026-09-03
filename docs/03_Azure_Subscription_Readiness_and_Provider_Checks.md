# Azure Subscription Readiness and Resource Provider Checks

## Purpose

This document records the commands and verified results used to prepare the Azure subscription for the Azure Local Jumpstart / LocalBox accreditation lab.

The checks are separated from deployment so that subscription, permissions, providers, region and quota can be verified before a new allocation. The existing lab has already been deployed successfully. This document was reconciled on 3 September 2026 with the [verified walkthrough](04_Azure_LocalBox_Lab_Walkthrough.md) and [dependency register](05_Azure_Local_Accreditation_Dependency_Register.md); it is not a new live quota or access check.

> **Active lab subscription:** `<LAB-SUBSCRIPTION-NAME>`
>
> **Target Azure region:** `Australia East` (`australiaeast`)
>
> **Deployed LocalBox host size:** `Standard_E32s_v5`
>
> The subscription ID is intentionally not stored in this public repository. Commands read it dynamically from the active Azure CLI context.

---

## 1. Confirm the Active Subscription

### What this command does

Displays the Azure subscription currently selected by Azure CLI.

### Why we run it

This is the first safety check. It prevents commands or deployments from running against the wrong subscription.

### Command

```powershell
az account show `
    --query "{Name:name,State:state,IsDefault:isDefault}" `
    -o table
```

### Verified result

The selected lab subscription was verified as Enabled before deployment. Its display name is represented by a placeholder in this public document; confirm the actual selection locally.

### Change impact

**Read-only.** No Azure resource is changed.

---

## 2. Check Effective RBAC Access

### What this check does

Finds Azure RBAC assignments for the signed-in user, including inherited roles and group-based assignments.

### Why we run it

In enterprise environments, access may come from a parent management group rather than a direct subscription assignment. LocalBox deployment still needs sufficient effective permissions.

### Commands

```powershell
$subId = az account show --query id -o tsv
$userId = az ad signed-in-user show --query id -o tsv

az role assignment list `
    --assignee $userId `
    --scope "/subscriptions/$subId" `
    --include-inherited `
    --include-groups `
    --query "[].{Role:roleDefinitionName,Scope:scope,Principal:principalName}" `
    -o table
```

### Verified result

The signed-in identity has an **Owner** role inherited from a parent management group. This provides effective Owner access to the lab subscription.

### Change impact

**Read-only.** No role assignment is created or changed.

---

## 3. Check Azure Local, Azure Arc and Compute Resource Providers

### What this check does

Reads the registration state of the Azure resource providers used by Azure Local, Azure Arc, Azure Local VM management, monitoring, supporting services and Azure Compute quota/SKU operations.

### Why we run it

Azure Local does not rely on a single Azure namespace. Different capabilities use different resource providers, so a fresh subscription must be checked before deployment. `Microsoft.Compute` is also required for VM operations and for reliable compute quota checks.

### Command

```powershell
$providers = @(
    "Microsoft.AzureStackHCI",
    "Microsoft.HybridCompute",
    "Microsoft.GuestConfiguration",
    "Microsoft.HybridConnectivity",
    "Microsoft.ResourceConnector",
    "Microsoft.Kubernetes",
    "Microsoft.KubernetesConfiguration",
    "Microsoft.HybridContainerService",
    "Microsoft.EdgeMarketplace",
    "Microsoft.ExtendedLocation",
    "Microsoft.Attestation",
    "Microsoft.Storage",
    "Microsoft.KeyVault",
    "Microsoft.Insights",
    "Microsoft.Compute"
)

foreach ($provider in $providers) {
    az provider show `
        --namespace $provider `
        --query "{Provider:namespace,Status:registrationState}" `
        -o table
}
```

### Initial result

The company subscription was effectively fresh. `Microsoft.GuestConfiguration` was already registered, while the remaining core Azure Local and Arc providers were not registered. `Microsoft.Compute` was subsequently validated as part of compute-readiness troubleshooting before quota evidence was collected.

### Change impact

**Read-only.** This command does not register a provider.

---

## 4. Provider Purpose Mapping

| Resource provider | Purpose in the Azure Local lab |
| --- | --- |
| `Microsoft.AzureStackHCI` | Core Azure Local resource registration and management. |
| `Microsoft.HybridCompute` | Azure Arc-enabled server and connected-machine representation. |
| `Microsoft.GuestConfiguration` | Machine configuration and Azure Policy guest-governance scenarios. |
| `Microsoft.HybridConnectivity` | Connectivity capabilities used by Azure Arc-enabled resources. |
| `Microsoft.ResourceConnector` | Azure Resource Bridge and Azure-based management of Azure Local resources. |
| `Microsoft.Kubernetes` | Azure Arc-enabled Kubernetes scenarios and related platform dependencies. |
| `Microsoft.KubernetesConfiguration` | Kubernetes extensions, configuration management and GitOps capabilities. |
| `Microsoft.HybridContainerService` | Required by the observed Azure Local Arc integration validation, even without an AKS demonstration. |
| `Microsoft.EdgeMarketplace` | Azure Local marketplace and platform dependency included in the verified deployment provider set. |
| `Microsoft.ExtendedLocation` | Custom Locations and Azure resources projected to an on-premises or edge location. |
| `Microsoft.Attestation` | Attestation-related deployment and security workflows where required. |
| `Microsoft.Storage` | Azure Storage resources used by deployment or operational workflows. |
| `Microsoft.KeyVault` | Secrets, certificates and deployment dependencies that use Azure Key Vault. |
| `Microsoft.Insights` | Azure Monitor, diagnostics and monitoring integration. |
| `Microsoft.Compute` | Azure VM operations, VM SKU discovery and regional / VM-family compute quota checks. |

The validated LocalBox deployment required `Microsoft.HybridContainerService` and `Microsoft.EdgeMarketplace`. Both reached Registered before validation succeeded, as recorded in the [dependency register](05_Azure_Local_Accreditation_Dependency_Register.md#4-mandatory-azure-local-resource-providers). Their registration is a deployment dependency and does not add an AKS demonstration to accreditation scope.

---

## 5. Register the Required Core Providers

### What this command does

Registers the Azure resource providers required for the current Azure Local and Azure Arc lab scope.

### Why we run it

The fresh subscription did not have the core Azure Local and Arc namespaces registered. These namespaces must be available before the related resource types can be deployed or managed.

### Command

```powershell
$providersToRegister = @(
    "Microsoft.AzureStackHCI",
    "Microsoft.HybridCompute",
    "Microsoft.HybridConnectivity",
    "Microsoft.ResourceConnector",
    "Microsoft.Kubernetes",
    "Microsoft.KubernetesConfiguration",
    "Microsoft.HybridContainerService",
    "Microsoft.EdgeMarketplace",
    "Microsoft.ExtendedLocation",
    "Microsoft.Attestation",
    "Microsoft.Storage",
    "Microsoft.KeyVault",
    "Microsoft.Insights",
    "Microsoft.Compute"
)

foreach ($provider in $providersToRegister) {
    Write-Host "Registering $provider ..."
    az provider register --namespace $provider
}
```

### Change impact

**Changes the Azure subscription configuration.** It registers resource-provider namespaces but does not create billable workload resources by itself.

---

## 6. Verify Provider Registration

### What this command does

Checks whether each requested provider has completed registration.

### Why we run it

Provider registration is asynchronous. The deployment should not proceed until the required namespaces reach the `Registered` state.

### Command

```powershell
foreach ($provider in $providersToRegister) {
    az provider show `
        --namespace $provider `
        --query "{Provider:namespace,Status:registrationState}" `
        -o table
}
```

### Verified result

The required deployment providers were confirmed as **Registered**. `Microsoft.GuestConfiguration` was already registered before the initial registration step. The later deployment validation identified and verified `Microsoft.HybridContainerService` and `Microsoft.EdgeMarketplace`; the command lists above now include both. This reconciles the initial readiness procedure with the final deployment evidence.

### Readiness status

**PASS.** Resource-provider readiness is complete for the current lab scope.

### Change impact

**Read-only.** The verification command does not modify Azure resources.

---

## 7. Locked Deployment Baseline

The deployed accreditation lab uses:

```text
Azure resource region        australiaeast
Azure Local instance region  australiaeast
LocalBox host size           Standard_E32s_v5
```

These values are verified in the walkthrough and dependency register. Use them consistently in accreditation-facing documentation. The successful deployment establishes historical host usability; it does not guarantee current spare quota or capacity for another allocation.

---

## 8. Check Australia East Compute Quota Before a New Allocation

For a new deployment or capacity change, check regional and relevant VM-family headroom in the active subscription.

**Read-only reference command; not rerun during this documentation audit:**

```powershell
az vm list-usage `
    --location australiaeast `
    -o table
```

Compare the requested allocation with current usage and limits. Do not reuse quota numbers from another region or treat a past deployment as a current capacity reservation.

**Evidence status:** the existing host deployment succeeded. Current spare quota values were not collected in this audit. No additional quota test is required solely to update this document or to repeat a completed accreditation deployment.

---

## 9. Check Host SKU Restrictions Before a New Allocation

For a fresh allocation, inspect the intended host SKU in the correct region.

**Read-only reference command; not rerun during this documentation audit:**

```powershell
az vm list-skus `
    --location australiaeast `
    --resource-type virtualMachines `
    --size Standard_E32s_v5 `
    --all `
    --query "[?name=='Standard_E32s_v5'].{Name:name,Locations:locations,Restrictions:restrictions}" `
    -o json
```

Review restrictions rather than treating inclusion in the response as permission to allocate. The `--all` option includes restricted SKUs. See [Microsoft's SKU troubleshooting guidance](https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/error-sku-not-available).

**Recorded result:** `Standard_E32s_v5` was successfully deployed in Australia East. A fresh capacity guarantee or new SKU response is not claimed here.

---

## 10. Check Azure CLI and Bicep Readiness

### What this check does

Confirms Azure CLI and Bicep are available before deployment.

### Commands

```powershell
az version
```

```powershell
az bicep version
```

### Verified result

```text
Azure CLI: 2.89.1
Bicep CLI: 0.46.1
```

Bicep was not initially present in the Cloud Shell session. It was installed with:

```powershell
az bicep install
```

The installation completed successfully and `az bicep version` returned Bicep CLI version `0.46.1`.

### Readiness status

**PASS.** Azure CLI and Bicep tooling required for the LocalBox deployment workflow are available.

### Change impact

`az version` and `az bicep version` are **read-only**. `az bicep install` changes only the Cloud Shell / Azure CLI client tooling and does not create or modify Azure workload resources.

---

## 11. Evidence to Capture

For the accreditation walkthrough, capture the following without exposing confidential identifiers or credentials:

1. Correct subscription context and Enabled state, with the display name sanitized for publication.
2. Effective Owner access confirmation.
3. Initial provider status.
4. Provider registration commands used.
5. Final provider registration status.
6. Australia East deployment configuration.
7. A dated quota result when a new allocation requires verification.
8. The recorded `Standard_E32s_v5` deployment, or a fresh SKU-restriction result for a new allocation.
9. Azure CLI and Bicep version checks.
10. Deployment outcome and its linked validation evidence.

The repository is public. Full subscription IDs, credentials, secrets and internal company information should not be committed unless they are explicitly safe for public disclosure.

---

## 12. Recorded Readiness and Current Position

| Check | Status and evidence boundary |
| --- | --- |
| Correct lab subscription and effective access | PASS at the recorded deployment checkpoint |
| Required Azure Local / Arc / Compute providers | PASS; final deployment dependencies included |
| Target region: Australia East | PASS; deployed baseline |
| `Standard_E32s_v5` host deployment | PASS; recorded in the walkthrough |
| Current spare quota / availability for another allocation | Not rechecked; verify when a new allocation or size change is needed |
| Azure CLI and Bicep readiness | PASS at the recorded tooling checkpoint |
| LocalBox deployment | COMPLETE; not awaiting a pre-deployment decision |
| Workload VM creation, network, guest management and start/stop/restart | PASS; see the VM validation record |
| Azure monitoring and management | NEXT / pending validation |
| Azure Local platform update/lifecycle | PENDING |

Continue with the monitoring prerequisite described in the [current recovery checkpoint](10_Lab_Recovery_Checkpoint_and_Follow_Up.md). Do not restart a pre-deployment GO / NO-GO review merely because this readiness procedure predates the completed deployment.

Readiness checks for a fresh deployment remain reusable; existing PASS entries describe recorded evidence and must not be presented as new live checks.
