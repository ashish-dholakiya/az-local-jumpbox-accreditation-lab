# AKS Evidence Manifest

> **Document Purpose**
>
> This manifest provides a complete inventory of all evidence collected
> during the implementation and validation of AKS enabled by Azure Arc
> on Azure Local.

------------------------------------------------------------------------

# Evidence Metadata

  Item                Value
  ------------------- ------------------------------------
  Project             Azure Local Mastery Lab
  Workload            AKS Enabled by Azure Arc
  Environment         Lab
  Validation Status   Verified -- Core Workload Complete

------------------------------------------------------------------------

# Evidence Inventory

  ----------------------------------------------------------------------------
  ID      Evidence            Type           Status          Used In
  ------- ------------------- -------------- --------------- -----------------
  EV-01   AKS Overview        Screenshot     Verified        Documentation,
                                                             Presentation

  EV-02   Node Pool           Screenshot     Verified        Documentation,
                                                             Presentation

  EV-03   Kubernetes          Screenshot     Verified        Documentation,
          Workloads                                          Presentation

  EV-04   Services &          Screenshot     Verified        Documentation,
          LoadBalancer                                       Presentation

  EV-05   Browser Validation  Screenshot     Verified        Documentation,
                                                             Presentation

  EV-06   Azure Local Health  CLI Output     Verified        Documentation

  EV-07   Extensions          Screenshot     Verified        Documentation

  EV-08   Resource Tags       Screenshot     Verified        Documentation

  EV-09   Monitoring Findings Screenshot +   Limited Scope   Documentation
                              Logs                           
  ----------------------------------------------------------------------------

------------------------------------------------------------------------

# Validation Summary

## Infrastructure

-   Azure Local cluster healthy
-   Cluster nodes operational
-   Storage healthy
-   Cloud Witness online

## Azure Arc

-   Connected
-   Operational

## AKS

-   Deployment succeeded
-   Node pool operational
-   Linux worker validated

## Kubernetes

-   Sample workload deployed
-   Ready 1/1
-   LoadBalancer operational

## Application

-   Browser validation successful
-   External IP reachable

------------------------------------------------------------------------

# Evidence Quality Checklist

-   Screenshots sanitized
-   Resource IDs removed where required
-   Validation completed after deployment
-   Evidence mapped to documentation
-   Evidence mapped to presentation

------------------------------------------------------------------------

# Known Limitation

Azure Monitor Containers extension deployed successfully.

Container Insights telemetry was not observed during the validation
period.

This finding has been documented separately and does not invalidate the
workload validation.

------------------------------------------------------------------------

# Approval Status

  Area             Status
  ---------------- -----------
  Infrastructure   Approved
  Azure Arc        Approved
  AKS              Approved
  Networking       Approved
  Workload         Approved
  Monitoring       Follow-up
  Documentation    Complete

------------------------------------------------------------------------

# Related Documents

-   README.md
-   12_AKS_Enabled_by_Azure_Arc_Minimal_PoC.md
-   12.1_AKS_Troubleshooting.md
-   12.2_AKS_Evidence_Guide.md
-   12.3_AKS_Presentation_Guide.md
-   12.4_AKS_Lessons_Learned.md
