# AKS Presentation Mapping

> **Purpose**
>
> This document maps every evidence artifact to the accreditation
> presentation. It ensures each slide is backed by verifiable technical
> evidence and provides guidance for the presenter.

------------------------------------------------------------------------

# Presentation Overview

  ------------------------------------------------------------------------
  Slide         Topic            Primary Evidence
  ------------- ---------------- -----------------------------------------
  1             Azure Local &    AKS Overview, Node Pool
                AKS Overview     

  2             Workload &       Workloads, Services, Browser Validation
                Networking       
                Validation       

  3             Monitoring &     Extensions, Monitoring Findings
                Lessons Learned  
  ------------------------------------------------------------------------

------------------------------------------------------------------------

# Slide 1 -- Azure Local & AKS Overview

## Objective

Demonstrate that AKS enabled by Azure Arc was successfully deployed on
Azure Local.

## Evidence

  Evidence ID   Artifact
  ------------- --------------------
  EV-01         AKS Overview
  EV-02         Node Pool
  EV-06         Azure Local Health

## Key Messages

-   Azure Arc Connected
-   AKS Provisioning Succeeded
-   Kubernetes v1.32.9
-   Linux Node Pool
-   Azure Local platform healthy

## Speaker Notes

Explain the deployment scope, platform architecture, and confirm that
Azure Local remained healthy before and after AKS deployment.

------------------------------------------------------------------------

# Slide 2 -- Workload & Networking Validation

## Objective

Demonstrate end-to-end workload deployment and application access.

## Evidence

  Evidence ID   Artifact
  ------------- ----------------------
  EV-03         Kubernetes Workloads
  EV-04         LoadBalancer Service
  EV-05         Browser Validation

## Key Messages

-   Namespace created
-   Workload Ready (1/1)
-   MetalLB assigned external IP
-   Application reachable via browser

## Demonstration Flow

Azure Local VM

↓

External IP

↓

MetalLB

↓

Kubernetes Service

↓

NGINX Pod

↓

Application Response

------------------------------------------------------------------------

# Slide 3 -- Monitoring & Lessons Learned

## Objective

Summarize operational readiness, monitoring status, and implementation
learnings.

## Evidence

  Evidence ID   Artifact
  ------------- ---------------------
  EV-07         Extensions
  EV-09         Monitoring Findings

## Key Messages

-   Arc Networking operational
-   Azure Monitor extension deployed
-   Telemetry validation pending
-   Platform ready for next phase

## Speaker Notes

Clearly distinguish successful monitoring integration from pending
telemetry validation. State that this limitation does not affect
workload validation.

------------------------------------------------------------------------

# Assessor Questions

  -----------------------------------------------------------------------
  Question                Suggested Response
  ----------------------- -----------------------------------------------
  Is AKS operational?     Yes. Verified through successful workload
                          deployment and browser validation.

  Why was CLI used?       Arc Networking deployment through the portal
                          did not complete in the lab. Azure CLI
                          completed successfully.

  Is monitoring complete? Extension deployment is complete; telemetry
                          validation remains a documented follow-up.

  Did AKS impact Azure    No. Azure Local health remained stable
  Local?                  throughout deployment.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Presentation Checklist

-   Use sanitized screenshots only.
-   Avoid subscription and tenant identifiers.
-   Do not include failed deployment screenshots.
-   Present evidence in deployment order.
-   End with lessons learned and next phase (AVD).

------------------------------------------------------------------------

# Final Outcome

The presentation demonstrates a successful AKS enabled by Azure Arc
implementation on Azure Local, supported by validated technical evidence
and documented operational observations.
