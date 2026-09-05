# 12. AKS Enabled by Azure Arc on Azure Local -- Minimal PoC

> **Status:** Verified -- Core Workload Complete\
> **Environment:** Azure Local Lab (Accreditation)

## 1. Objective

This document describes the complete implementation journey of deploying
**AKS enabled by Azure Arc** on an Azure Local cluster using the Azure
portal, validating workload deployment, configuring MetalLB, and
documenting the evidence collected during the accreditation lab.

------------------------------------------------------------------------

## 2. Lab Architecture

-   Azure Subscription
-   Azure Local (2-node cluster)
-   Azure Arc
-   Custom Location
-   AKS enabled by Azure Arc
-   One Linux node pool
-   Arc Networking (MetalLB)
-   Azure Monitor Containers
-   Log Analytics Workspace

------------------------------------------------------------------------

## 3. Environment Summary

  Component             Value
  --------------------- --------------------------
  Azure Local Cluster   2 Nodes
  Kubernetes Version    1.32.9
  Worker Node Pool      Linux
  VM Size               Standard_A4_v2
  Control Plane         1 Node
  Worker Nodes          1
  Network               localbox-vm-lnet-vlan200

------------------------------------------------------------------------

## 4. Deployment Approach

Deployment was performed primarily through the Azure Portal to
understand every configuration page and validation step. CLI was used
only where Microsoft portal functionality was insufficient (MetalLB
configuration).

------------------------------------------------------------------------

## 5. Azure Portal Configuration

### Resource Group

-   rg-azlocal-localbox-accreditation-aue

### Custom Location

-   jumpstart

### Cluster

-   aksarc-localbox-poc

### Node Pool

-   Linux
-   Standard_A4_v2
-   Node Count = 1

------------------------------------------------------------------------

## 6. Networking

Logical Network

-   localbox-vm-lnet-vlan200

Load Balancer

-   Arc Networking Extension
-   MetalLB
-   Advertise Mode = ARP

Address Pool

192.168.200.240 -- 192.168.200.249

------------------------------------------------------------------------

## 7. Sample Workload

Namespace

-   accreditation

Deployment

-   accreditation-web

Container

-   nginx

Service

-   LoadBalancer

External IP

-   192.168.200.240

------------------------------------------------------------------------

## 8. Validation Results

### Azure Local

-   Cluster Healthy
-   Storage Healthy
-   Cloud Witness Online
-   Virtual Disks Healthy
-   AKS Control Plane VM Online
-   AKS Worker VM Online

### Kubernetes

-   Deployment Ready 1/1
-   Service Status OK
-   External IP Assigned
-   Application Reachable

### Application

Validation URL

http://192.168.200.240

Result

Welcome to nginx!

------------------------------------------------------------------------

## 9. Monitoring

Container monitoring was enabled during deployment.

Azure Monitor Containers extension:

Status: Succeeded

Observation:

Container Insights telemetry was not observed during the validation
window.

This does not affect the successful deployment and validation of the AKS
workload.

------------------------------------------------------------------------

## 10. Evidence Collected

-   AKS Overview
-   Node Pool
-   Kubernetes Workloads
-   LoadBalancer Service
-   External IP Assignment
-   NGINX Browser Validation
-   Azure Local Cluster Health
-   Extensions
-   Tags

------------------------------------------------------------------------

## 11. Lessons Learned

-   Validate Azure Local health before and after AKS deployment.
-   Portal deployment is useful for learning every configuration.
-   MetalLB configuration through CLI is more reliable than the portal
    in this lab.
-   Keep a dedicated IP range for LoadBalancer services.
-   Separate monitoring validation from workload validation.

------------------------------------------------------------------------

## 12. Known Limitations

-   Container Insights telemetry pending.
-   Cloud Shell cannot directly reach the Kubernetes API endpoint hosted
    on the Azure Local logical network.
-   Local validation should be performed from a workload VM.

------------------------------------------------------------------------

## 13. Next Phase

-   Azure Virtual Desktop
-   Azure Monitor telemetry validation
-   GitOps
-   Ingress Controller
-   Production-ready HA design

------------------------------------------------------------------------

## 14. Accreditation Conclusion

The AKS enabled by Azure Arc proof of concept was successfully deployed
and validated on Azure Local.

The following capabilities were successfully demonstrated:

-   Azure Arc integration
-   AKS deployment
-   Linux node pool
-   MetalLB LoadBalancer
-   Sample workload deployment
-   External application access
-   Azure Local platform stability

Monitoring integration was configured successfully; telemetry validation
remains a follow-up activity and is outside the core workload validation
scope.
