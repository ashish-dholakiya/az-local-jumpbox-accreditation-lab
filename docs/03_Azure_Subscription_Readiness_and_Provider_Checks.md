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

## 3. Check Azure Local and Azure Arc Resource Providers

### What this check does

Reads the registration state of the Azure resource providers used by Azure Local, Azure Arc, Azure Local VM management, monitoring and supporting services.

### Why we run it

Azure Local does not rely on a single Azure namespace. Different capabilities use different resource providers, so a fresh subscription must be checked before deployment.

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
    "Microsoft.Insights"
)

foreach ($provider in $providers) {
    az provider show `
        --namespace $provider `
        --query "{Provider:namespace,Status:registrationState}" `
        -o table
}
```

### Initial result

The company subscription was effectively fresh. `Microsoft.GuestConfiguration` was already registered, while the remaining core providers were not registered.

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
    "Microsoft.Insights"
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

The following providers were confirmed as **Registered** on the company lab subscription:

```text
Microsoft.AzureStackHCI               Registered
Microsoft.HybridCompute               Registered
Microsoft.HybridConnectivity          Registered
Microsoft.ResourceConnector           Registered
Microsoft.Kubernetes                  Registered
Microsoft.KubernetesConfiguration     Registered
Microsoft.ExtendedLocation            Registered
Microsoft.Attestation                 Registered
Microsoft.Storage                     Registered
Microsoft.KeyVault                    Registered
Microsoft.Insights                    Registered
```

`Microsoft.GuestConfiguration` was already registered before the registration step.

### Readiness status

**PASS.** Core Azure Local and Azure Arc resource-provider readiness is complete for the current lab scope.

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

LocalBox requires a relatively large Azure VM. Even when permissions and providers are ready, insufficient regional or VM-family quota can block deployment.

### Command

```powershell
az vm list-usage `
    --location centralindia `
    -o table
```

### What to review

Pay particular attention to:

- Total Regional vCPUs
- The VM-family quota for the LocalBox VM SKU
- Current regional usage
- Remaining vCPU headroom

### Change impact

**Read-only.** No Azure resource is changed.

---

## 9. Check LocalBox VM SKU Availability in Central India

### What this command does

Checks whether candidate LocalBox VM SKUs are available in Central India and whether Azure reports any subscription restrictions.

### Why we run it

Quota alone is not enough. The selected VM SKU must also be available to this subscription in the chosen region.

### Commands

```powershell
az vm list-skus `
    --location centralindia `
    --size Standard_E32s_v6 `
    --all `
    --query "[].{Name:name,Restrictions:restrictions}" `
    -o json
```

Compare with:

```powershell
az vm list-skus `
    --location centralindia `
    --size Standard_E32s_v5 `
    --all `
    --query "[].{Name:name,Restrictions:restrictions}" `
    -o json
```

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

### Change impact

**Read-only.** No Azure resource is changed.

---

## 11. Evidence to Capture

For the accreditation walkthrough, capture the following without exposing confidential identifiers or credentials:

1. Active company subscription name and Enabled state.
2. Effective Owner access confirmation.
3. Initial provider status.
4. Provider registration commands used.
5. Final provider registration status.
6. Central India region decision.
7. Central India quota output.
8. LocalBox VM SKU availability result.
9. Azure CLI and Bicep version checks.
10. Final deployment readiness decision.

The repository is public. Full subscription IDs, credentials, secrets and internal company information should not be committed unless they are explicitly safe for public disclosure.

---

## 12. Current Readiness Status

| Check | Status |
| --- | --- |
| Correct company subscription active | PASS |
| Effective Owner access | PASS |
| Required Azure Local / Arc providers | PASS |
| Target region selected: Central India | PASS |
| Central India compute quota | PENDING |
| LocalBox VM SKU availability | PENDING |
| Azure CLI readiness | PENDING |
| Bicep readiness | PENDING |
| Final LocalBox deployment decision | PENDING |

The next implementation step is to validate **Central India compute quota and LocalBox VM SKU availability** before any billable deployment starts.
