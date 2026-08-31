# Azure Subscription Readiness and Resource Provider Checks

## Purpose

This document records the commands and verified results used to prepare the Azure subscription for the Azure Local Jumpstart / LocalBox accreditation lab.

The checks are intentionally separated from the deployment steps so that subscription, permissions, provider registration, region and quota issues are identified before billable lab resources are created.

> **Active lab subscription:** `ME-MngEnvMCAP085303-v-dholakiyaa-1`
>
> **Target Azure region:** `Central India`
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
    --query "{Name:name,SubscriptionId:id,State:state,IsDefault:isDefault}" `
    -o table
```

### Verified result

The active company subscription is `ME-MngEnvMCAP085303-v-dholakiyaa-1` and its state is `Enabled`.

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
| `Microsoft.ExtendedLocation` | Custom Locations and Azure resources projected to an on-premises or edge location. |
| `Microsoft.Attestation` | Attestation-related deployment and security workflows where required. |
| `Microsoft.Storage` | Azure Storage resources used by deployment or operational workflows. |
| `Microsoft.KeyVault` | Secrets, certificates and deployment dependencies that use Azure Key Vault. |
| `Microsoft.Insights` | Azure Monitor, diagnostics and monitoring integration. |
| `Microsoft.Compute` | Azure VM operations, VM SKU discovery and regional / VM-family compute quota checks. |

`Microsoft.HybridContainerService` is not part of the current core registration set because the accreditation scope does not currently require an AKS-on-Azure-Local demonstration. It can be enabled later if the lab scope expands.

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

The required Azure Local and Azure Arc providers were confirmed as **Registered** on the company lab subscription. `Microsoft.GuestConfiguration` was already registered before the registration step. `Microsoft.Compute` was also made available before compute quota validation.

### Readiness status

**PASS.** Resource-provider readiness is complete for the current lab scope.

### Change impact

**Read-only.** The verification command does not modify Azure resources.

---

## 7. Lock the Deployment Region

### Decision

The accreditation lab will use:

```text
Central India
```

### Why this matters

VM SKU availability, subscription quota and deployment support are evaluated per Azure region. All subsequent quota and LocalBox SKU checks therefore use `centralindia`.

---

## 8. Check Central India Compute Quota

### What this command does

Displays Azure Compute quota and current usage for Central India.

### Why we run it

LocalBox uses a large Azure VM. Even when permissions and providers are ready, insufficient regional or VM-family quota can block deployment.

### Command

```powershell
az vm list-usage `
    --location centralindia `
    -o table
```

### Verified result

The company lab subscription currently has the following Central India compute capacity:

```text
Total Regional vCPUs              Used: 0    Limit: 100
Standard Esv6 Family vCPUs        Used: 0    Limit: 100
```

The planned `Standard_E32s_v6` LocalBox host requires 32 vCPUs. The subscription therefore has sufficient headroom in both the regional quota and the Esv6 family quota.

### Readiness status

**PASS.** No compute quota increase is currently required for a single 32-vCPU LocalBox host in Central India.

### Change impact

**Read-only.** No Azure resource is changed.

---

## 9. Check LocalBox VM SKU Availability in Central India

### What this command does

Checks whether the candidate LocalBox VM SKU is available in Central India.

### Why we run it

Quota alone is not enough. The selected VM size must also be available in the target region.

### Lightweight validation command used during the lab

```powershell
az vm list-sizes `
    --location centralindia `
    --query "[?name=='Standard_E32s_v6']" `
    -o table
```

> `az vm list-sizes` is deprecated, but it returned a valid lightweight SKU-availability check when `az vm list-skus --all` was slow in Cloud Shell. For future runs, prefer `az vm list-skus` where practical.

### Verified result

`Standard_E32s_v6` was returned in Central India with:

```text
Name:              Standard_E32s_v6
NumberOfCores:     32
MemoryInMB:        262144
MaxDataDiskCount:  64
```

### Readiness status

**PASS.** `Standard_E32s_v6` is visible in Central India for the current subscription context.

### Change impact

**Read-only.** No Azure resource is created.

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

1. Active company subscription name and Enabled state.
2. Effective Owner access confirmation.
3. Initial provider status.
4. Provider registration commands used.
5. Final provider registration status.
6. Central India region decision.
7. Central India quota result.
8. `Standard_E32s_v6` availability result.
9. Azure CLI and Bicep version checks.
10. Final deployment readiness decision.

The repository is public. Full subscription IDs, credentials, secrets and internal company information should not be committed unless they are explicitly safe for public disclosure.

---

## 12. Current Readiness Status

| Check | Status |
| --- | --- |
| Correct company subscription active | PASS |
| Effective Owner access | PASS |
| Required Azure Local / Arc / Compute providers | PASS |
| Target region selected: Central India | PASS |
| Central India compute quota | PASS |
| `Standard_E32s_v6` availability | PASS |
| Azure CLI readiness | PASS |
| Bicep readiness | PASS |
| Final LocalBox deployment decision | PENDING |

All subscription, permission, provider, region, quota, SKU and client-tooling readiness checks completed so far are **PASS**. The next implementation step is the final LocalBox pre-deployment GO / NO-GO review before any billable lab resources are created.
