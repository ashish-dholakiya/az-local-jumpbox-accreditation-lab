# Azure Subscription Readiness and Resource Provider Checks

## Purpose

This document records the commands used to prepare and validate the Azure subscription for the Azure Local Jumpstart / LocalBox accreditation lab.

The checks are kept separate from the deployment steps so that subscription, permissions, provider registration, region and quota issues can be identified before any billable lab resources are created.

> **Active lab subscription:** `ME-MngEnvMCAP085303-v-dholakiyaa-1`
>
> **Target Azure region:** `Central India`
>
> The subscription ID is intentionally not stored in this public repository. Commands read it dynamically from the active Azure CLI context.

---

## 1. List Available Azure Subscriptions

### What this command does

Lists the Azure subscriptions available to the signed-in identity.

### Why we run it

This is the first safety check. It confirms that Azure CLI is using the intended company-provided subscription before any provider registration or deployment activity begins.

### Command

```powershell
az account list `
    --query "[].{Name:name,SubscriptionId:id,State:state,IsDefault:isDefault}" `
    -o table
```

### Expected result

The active lab subscription should appear as:

```text
ME-MngEnvMCAP085303-v-dholakiyaa-1
```

The subscription state should be `Enabled`.

### Change impact

**Read-only.** No Azure resource is changed.

---

## 2. Confirm the Active Subscription

### What this command does

Displays the Azure subscription currently selected by Azure CLI.

### Why we run it

Even when only one subscription is visible, we explicitly verify the active context before continuing.

### Command

```powershell
az account show `
    --query "{Name:name,SubscriptionId:id,State:state,IsDefault:isDefault}" `
    -o table
```

### Expected result

The subscription name should be `ME-MngEnvMCAP085303-v-dholakiyaa-1` and the state should be `Enabled`.

### Change impact

**Read-only.** No Azure resource is changed.

---

## 3. Check Effective RBAC Access

### What this check does

Finds the effective Azure RBAC assignments for the signed-in user, including roles inherited from a management group and roles granted through group membership.

### Why we run it

A direct subscription-level assignment may not exist in enterprise environments. Access can be inherited from a parent management group. The LocalBox deployment identity must still have sufficient effective permissions to register providers, create resources and perform deployment operations.

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

### Actual lab result

The signed-in identity has an **Owner** role inherited from a parent management group. This is treated as effective Owner access for the subscription.

### Change impact

**Read-only.** No role assignment is created or changed.

---

## 4. Check Azure Local and Azure Arc Resource Providers

### What this check does

Reads the registration state of the Azure resource providers used by Azure Local, Azure Arc, Azure Local VM management, monitoring and supporting services.

### Why we run it

Azure Local does not depend on a single resource provider. Different capabilities use different Azure namespaces. A fresh subscription may have only a small number of providers registered by default.

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

### Actual initial lab result

This company subscription was effectively fresh. `Microsoft.GuestConfiguration` was already registered, while the remaining core providers were not registered.

### Change impact

**Read-only.** This command does not register any provider.

---

## 5. What Each Provider Is Used For

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

`Microsoft.HybridContainerService` is intentionally not part of the current core registration set because the accreditation lab does not currently require an AKS-on-Azure-Local demonstration. It can be added later if the lab scope expands.

---

## 6. Register the Required Core Providers

### What this command does

Registers the Azure resource providers needed for the current Azure Local / Arc lab scope.

### Why we run it

The fresh subscription did not have the core Azure Local and Arc providers registered. Registration is required before the related Azure resource types can be deployed or managed.

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

### Expected result

Provider registration is asynchronous. The first result may show `Registering`.

### Change impact

**Changes the Azure subscription configuration.** It registers resource-provider namespaces but does not create billable workload resources by itself.

---

## 7. Verify Provider Registration

### What this command does

Checks whether each requested resource provider has completed registration.

### Why we run it

We do not continue to LocalBox deployment until the required providers reach the `Registered` state.

### Command

```powershell
foreach ($provider in $providersToRegister) {
    az provider show `
        --namespace $provider `
        --query "{Provider:namespace,Status:registrationState}" `
        -o table
}
```

### Expected result

Each provider in the required set should eventually show:

```text
Registered
```

### Change impact

**Read-only.** No provider or Azure resource is changed.

---

## 8. Lock the Deployment Region

### Decision

The accreditation lab will use:

```text
Central India
```

### Why this matters

VM SKU availability, subscription quota and deployment support are evaluated per Azure region. All subsequent quota and LocalBox SKU checks must therefore use `centralindia`.

---

## 9. Check Central India Compute Quota

### What this command does

Displays the current Azure Compute quota and usage for Central India.

### Why we run it

LocalBox requires a relatively large Azure VM. Even when the subscription has permission to deploy, insufficient regional or VM-family quota can block deployment.

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

## 10. Check LocalBox VM SKU Availability in Central India

### What this command does

Checks whether the candidate LocalBox VM SKUs are available in Central India and whether Azure reports any restrictions.

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

## 11. Check Azure CLI and Bicep Readiness

### What this check does

Confirms the Azure CLI and Bicep components are available before deployment.

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

## 12. Evidence to Capture

For the accreditation walkthrough, capture the following without exposing confidential identifiers or credentials:

1. Active company subscription name and Enabled state.
2. Effective Owner access confirmation.
3. Initial resource-provider status.
4. Provider registration commands used.
5. Final provider registration status.
6. Central India region decision.
7. Central India quota output.
8. LocalBox VM SKU availability result.
9. Azure CLI and Bicep version checks.
10. Final deployment readiness decision.

The repository is public. Full subscription IDs, tenant-sensitive details, credentials, secrets and internal company information should not be committed unless they are explicitly safe for public disclosure.

---

## 13. Readiness Decision

The subscription should be marked **GO** for LocalBox deployment only when all of the following are confirmed:

- Correct company subscription is active.
- Effective Owner access is available.
- Required resource providers are Registered.
- Central India is selected as the deployment region.
- Required LocalBox VM SKU is available in Central India.
- Regional and VM-family quota is sufficient.
- Azure CLI and Bicep checks pass.

Only after these checks should the actual LocalBox deployment begin.
