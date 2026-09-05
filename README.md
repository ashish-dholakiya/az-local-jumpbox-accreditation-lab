# Azure Local JumpBox Accreditation Lab

> **Enterprise Repository for Azure Local Accreditation, Azure Arc, AKS
> Enabled by Azure Arc, Architecture, Validation, Evidence, and Customer
> Deliverables**

------------------------------------------------------------------------

# Overview

This repository contains the complete implementation, validation,
evidence, and customer-facing deliverables for the **Azure Local JumpBox
Accreditation Lab** based on the **Contoso Manufacturing** scenario.

The repository is maintained as the **single source of truth** for all
technical implementation, documentation, operational evidence, and
accreditation deliverables.

------------------------------------------------------------------------

# Repository Objectives

-   Customer Discovery
-   Azure Local Architecture
-   Azure Local JumpBox / LocalBox Deployment
-   Azure Arc Integration
-   AKS Enabled by Azure Arc
-   Validation & Evidence Collection
-   VMware Migration Planning
-   Executive Presentation Material
-   Accreditation Knowledge Validation

------------------------------------------------------------------------

# Governing Source of Truth

The primary governing document is:

`docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md`

Every implementation, document, diagram, presentation, and evidence
package must remain traceable to this document.

------------------------------------------------------------------------

# Repository Structure

## Documentation

  Document   Purpose
  ---------- -----------------------------------------
  00         Accreditation Scope & Customer Scenario
  01         Customer Discovery & Assessment
  02         Azure Local Architecture
  03         Azure Subscription Readiness
  04         Azure LocalBox Walkthrough
  05         Dependency Register
  06         Azure Local Deployment Runbook
  07         Production Concepts
  08         WAC, Storage & Disk Validation
  09         VM, Network & Lifecycle Validation
  10         Recovery Checkpoint
  11         Monitoring & Platform Update Validation
  12         AKS Enabled by Azure Arc
  12.1       AKS Troubleshooting
  12.2       AKS Evidence Guide
  12.3       AKS Presentation Guide
  12.4       AKS Lessons Learned

------------------------------------------------------------------------

# Evidence Structure

``` text
evidence/
├── portal/
│   ├── screenshots
│   └── README.md
│
└── aks/
    ├── README.md
    ├── Evidence-Manifest.md
    ├── Screenshot-Index.md
    ├── Validation-Summary.md
    ├── Monitoring-Findings.md
    ├── Presentation-Mapping.md
    └── screenshots/
```

------------------------------------------------------------------------

# Diagrams

Repository architecture diagrams are stored under:

``` text
diagrams/
```

These diagrams represent the customer architecture and supporting Azure
Local design documentation.

------------------------------------------------------------------------

# Current Lab Status

  Area                           Status
  ------------------------------ --------------
  Azure Local                    ✅ Complete
  Azure Arc                      ✅ Complete
  Azure Local Platform Update    ✅ Complete
  Azure Local Validation         ✅ Complete
  AKS Enabled by Azure Arc       ✅ Complete
  Arc Networking                 ✅ Complete
  MetalLB Load Balancer          ✅ Complete
  Sample Kubernetes Workload     ✅ Complete
  Browser Validation             ✅ Complete
  Azure Monitor Extension        ✅ Installed
  Container Insights Telemetry   ⏳ Follow-up
  Documentation                  ✅ Complete
  Evidence Package               ✅ Complete

------------------------------------------------------------------------

# Accreditation Delivery Status

Current implementation includes:

-   Customer Discovery
-   Azure Local Architecture
-   Azure Subscription Readiness
-   Azure Local Deployment
-   Azure Arc Integration
-   Azure Local Validation
-   Platform Update Validation
-   AKS Enabled by Azure Arc
-   Evidence Collection

Presentation refinement, customer rehearsal, and knowledge validation
remain final accreditation activities.

------------------------------------------------------------------------

# Working Principles

Every implementation follows the sequence:

**Source of Truth → Design → Implementation → Validation → Evidence →
Documentation → Customer Explanation**

------------------------------------------------------------------------

# Public Repository Security

Never commit:

-   Passwords
-   Secrets
-   Certificates
-   Private Keys
-   Terraform State
-   Connection Strings
-   Customer confidential information
-   Unsanitized screenshots

------------------------------------------------------------------------

# Quick Navigation

## Core Azure Local

-   Architecture
-   Deployment
-   Validation
-   Monitoring

## AKS

-   Implementation Guide
-   Troubleshooting
-   Evidence
-   Presentation
-   Lessons Learned

## Evidence

-   Portal Validation
-   AKS Validation
-   Screenshots
-   Operational Evidence

------------------------------------------------------------------------

# Roadmap

Next implementation phases:

1.  Azure Virtual Desktop (AVD)
2.  GitOps
3.  Advanced Azure Monitor
4.  Production Operations
5.  Customer Presentation Finalization

------------------------------------------------------------------------

# Repository Governance

The **main** branch is the authoritative branch.

Documentation should only represent validated work.

Planned work must always be identified as planned.

------------------------------------------------------------------------

# License

This repository is intended for Azure Local accreditation, learning,
demonstration, and customer architecture discussions.

All Microsoft trademarks and product names remain the property of
Microsoft.
