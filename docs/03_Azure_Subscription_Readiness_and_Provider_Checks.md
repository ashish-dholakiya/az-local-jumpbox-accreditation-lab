# Azure Subscription Readiness and Resource Provider Checks

## Purpose

This document records the commands used to validate whether the Azure subscription is ready for the Azure Local Jumpstart / LocalBox lab.

The checks are intentionally separated from the deployment steps so that subscription issues can be identified before resources are created. Most commands in this document are read-only unless explicitly marked otherwise.

> **Lab subscription name:** `AD_Test_Platform`
>
> The subscription ID is intentionally not stored in this public repository. Use the correct subscription ID from your secure local notes or Azure portal when running the commands.

---

## 1. Confirm the Active Azure Account

### What this command does

Displays the Azure account and subscription that Azure CLI is currently using.

### Why we run it

Before running any subscription-level check, we need to make sure Azure CLI is authenticated with the expected tenant and subscription.

### Command

```powershell
az account show -o table
```

### Expected result

The output should show the intended account and subscription. For this lab, the subscription name should be:

```text
AD_Test_Platform
```

### Change impact

**Read-only.** This command does not change any Azure resource.

---

## 2. Select the Lab Subscription

### What this command does

Sets the Azure CLI context to the subscription that will be used for the lab.

### Why we run it

Azure CLI can have access to several subscriptions. Explicitly selecting the lab subscription helps prevent checks or deployments from running in the wrong subscription.

### Command

```powershell
$subscriptionId = "<your-subscription-id>"
az account set --subscription $subscriptionId
```

### Validation

Run:

```powershell
az account show --query "{Subscription:name, SubscriptionId:id, Tenant:tenantId, State:state}" -o table
```

### Expected result

The subscription should be `AD_Test_Platform` and the state should be `Enabled`.

### Change impact

`az account set` changes only the local Azure CLI context. It does not modify Azure resources.

---

## 3. Check Azure Local and Azure Arc Resource Providers

### What this check does

Reads the registration state of the Azure resource providers commonly used by Azure Local, Azure Arc, Azure Local VM management, monitoring, governance and optional Kubernetes services.

### Why we run it

Azure Local is not supported by a single Azure resource provider. Different capabilities depend on different providers. Checking them before deployment helps identify missing prerequisites early.

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
    "Microsoft.HybridContainerService",
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

### Expected result

Each provider will normally show one of the following states:

- `Registered`
- `Registering`
- `NotRegistered`

At this stage, the command is used only to collect the current state. Providers should not be registered blindly. Registration should be based on the feature being deployed and the current Microsoft requirements for that workflow.

### Change impact

**Read-only.** This command does not register or modify any provider.

---

## 4. What Each Provider Is Used For

| Resource provider | Primary purpose in this lab or Azure Local environment |
| --- | --- |
| `Microsoft.AzureStackHCI` | Core Azure Local resource provider used for Azure Local registration and management. |
| `Microsoft.HybridCompute` | Supports Azure Arc-enabled servers and connected-machine representation in Azure. |
| `Microsoft.GuestConfiguration` | Used for guest configuration, Azure Policy scenarios and machine-level governance. |
| `Microsoft.HybridConnectivity` | Supports connectivity capabilities associated with Azure Arc-enabled resources. |
| `Microsoft.ResourceConnector` | Used by Azure Resource Bridge scenarios and Azure-based management of resources running on Azure Local. |
| `Microsoft.Kubernetes` | Required for Azure Arc-enabled Kubernetes scenarios when Kubernetes is part of the solution. |
| `Microsoft.KubernetesConfiguration` | Supports Kubernetes extensions, configuration management and GitOps scenarios. |
| `Microsoft.ExtendedLocation` | Supports Custom Locations and Azure resources that are projected to an on-premises location. |
| `Microsoft.HybridContainerService` | Used for AKS on Azure Local scenarios. It is not required simply to run Windows or Linux VMs. |
| `Microsoft.Attestation` | Supports attestation-related Azure Local deployment and security workflows where required. |
| `Microsoft.Storage` | Supports Azure storage resources that may be required by deployment, migration or operational workflows. |
| `Microsoft.KeyVault` | Used when secrets, certificates or deployment dependencies require Azure Key Vault. |
| `Microsoft.Insights` | Supports monitoring, diagnostics and Azure Monitor integration. |

---

## 5. Create a Compact Provider Status Report

### What this command does

Produces a single table showing all providers and their registration state.

### Why we run it

The earlier loop is easy to read during troubleshooting, while this version is more suitable for evidence capture and documentation.

### Command

```powershell
$providerStatus = foreach ($provider in $providers) {
    $result = az provider show `
        --namespace $provider `
        --query "{Provider:namespace,Status:registrationState}" `
        -o json | ConvertFrom-Json

    [PSCustomObject]@{
        Provider = $result.Provider
        Status   = $result.Status
    }
}

$providerStatus | Format-Table -AutoSize
```

### Expected result

A table similar to:

```text
Provider                              Status
--------                              ------
Microsoft.AzureStackHCI               Registered
Microsoft.HybridCompute               Registered
Microsoft.GuestConfiguration          Registered
...
```

### Change impact

**Read-only.** No Azure resources are changed.

---

## 6. Check Subscription Role Assignment

### What this check does

Lists role assignments for the currently signed-in user at subscription scope.

### Why we run it

LocalBox deployment requires sufficient subscription-level permissions. The deployment documentation should be checked for the current required role before starting the lab. For this lab, we will verify whether the deployment identity has the required subscription-level access before creating resources.

### Commands

First identify the signed-in user:

```powershell
az account show --query user -o json
```

Then capture the signed-in username:

```powershell
$currentUser = az account show --query user.name -o tsv
$currentUser
```

Check role assignments at subscription scope:

```powershell
$subscriptionId = az account show --query id -o tsv

az role assignment list `
    --assignee $currentUser `
    --scope "/subscriptions/$subscriptionId" `
    --include-inherited `
    --query "[].{Role:roleDefinitionName,Scope:scope}" `
    -o table
```

### Expected result

The output should confirm that the deployment identity has the level of access required by the current LocalBox deployment guidance.

### Change impact

**Read-only.** No role assignment is created or modified.

---

## 7. Check Azure CLI and Bicep Readiness

### What this check does

Confirms the installed Azure CLI version and verifies whether the Bicep component is available.

### Why we run it

LocalBox uses Azure deployment automation. Verifying the client tools first avoids deployment failures caused by an outdated or missing local component.

### Commands

```powershell
az version
```

Check Bicep:

```powershell
az bicep version
```

### Expected result

Both commands should return version information without an error.

### Change impact

**Read-only.** No Azure resources are changed.

---

## 8. Evidence to Capture

For the accreditation walkthrough, capture the following results without exposing secrets or unnecessary subscription identifiers:

1. Active subscription name and state.
2. Tenant information only where needed for technical evidence.
3. Resource provider registration-state table.
4. Required deployment role confirmation.
5. Azure CLI version.
6. Bicep version.
7. Any missing prerequisite identified before deployment.

Sensitive information should be redacted before screenshots or outputs are committed to this public repository.

---

## 9. Next Step After These Checks

Once the commands above are completed, classify each provider as:

- **Required now and already registered**
- **Required now and needs registration**
- **Optional for the current lab**
- **Not required for the current accreditation scope**

Only after this review should any missing resource provider be registered.

The next implementation stage is the LocalBox deployment readiness decision: **GO** or **NO-GO**.
