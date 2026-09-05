# AKS Monitoring Findings

> **Purpose**
>
> This document records the monitoring implementation, validation
> activities, observations, findings, limitations, and recommended
> follow-up actions for AKS enabled by Azure Arc on Azure Local.

------------------------------------------------------------------------

# 1. Monitoring Scope

The monitoring validation covered:

-   Azure Monitor Containers Extension
-   Log Analytics Workspace integration
-   Container Insights
-   Kubernetes telemetry
-   Extension provisioning status

------------------------------------------------------------------------

# 2. Monitoring Components

  Component                            Status
  ------------------------------------ --------------------
  Azure Monitor Containers Extension   Succeeded
  Log Analytics Workspace              Configured
  Arc Extension                        Operational
  Container Insights Telemetry         Follow-up Required

------------------------------------------------------------------------

# 3. Validation Activities

The following validation activities were performed:

-   Verified extension deployment.
-   Confirmed extension provisioning state.
-   Opened Container Insights from Azure Portal.
-   Executed Log Analytics validation queries.
-   Reviewed Azure Monitor resources.

------------------------------------------------------------------------

# 4. Log Analytics Validation

Queries executed included:

-   Heartbeat
-   KubeNodeInventory

Observation:

No records were returned during the validation window.

------------------------------------------------------------------------

# 5. Findings

## Successful Findings

-   Azure Monitor Containers extension deployed successfully.
-   Extension provisioning completed without errors.
-   Log Analytics workspace association completed.

## Outstanding Observation

Container Insights telemetry was not available during the validation
period.

------------------------------------------------------------------------

# 6. Impact Assessment

This observation impacts:

-   Container performance dashboards
-   Node inventory dashboards
-   Historical telemetry analysis

This observation does **not** impact:

-   AKS deployment
-   Kubernetes workloads
-   Application accessibility
-   LoadBalancer functionality
-   Azure Local platform health

------------------------------------------------------------------------

# 7. Root Cause Assessment

During the implementation no infrastructure fault was identified.

The most likely causes include:

-   Initial telemetry ingestion delay
-   Data collection latency
-   Additional monitoring configuration requirements

Further investigation is recommended before production use.

------------------------------------------------------------------------

# 8. Recommendation

Before production deployment:

-   Validate telemetry ingestion.
-   Confirm Log Analytics data collection.
-   Verify Container Insights dashboards.
-   Perform long-duration monitoring validation.

------------------------------------------------------------------------

# 9. Evidence

Supporting evidence:

-   Azure Monitor extension status
-   Extension provisioning state
-   Log Analytics query results
-   Azure Portal screenshots

------------------------------------------------------------------------

# 10. Final Monitoring Status

  Area                    Status
  ----------------------- ---------------------
  Extension Deployment    PASS
  Workspace Association   PASS
  Telemetry Validation    FOLLOW-UP
  Operational Readiness   PARTIALLY VALIDATED

------------------------------------------------------------------------

# 11. Conclusion

Monitoring integration was successfully configured as part of the AKS
deployment.

Telemetry validation remains a follow-up activity and has been
documented as a known limitation. This finding does not invalidate the
successful deployment and validation of the AKS workload.
