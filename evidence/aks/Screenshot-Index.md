# AKS Screenshot Index

> **Purpose**
>
> This document provides a master index of all screenshots collected
> during the AKS enabled by Azure Arc implementation. It maps each
> screenshot to its purpose, documentation reference, and presentation
> usage.

------------------------------------------------------------------------

# Screenshot Naming Standard

All screenshots should follow this format:

    aksarc-<area>-<description>-redacted.png

Examples:

-   aksarc-cluster-overview-redacted.png
-   aksarc-nodepool-redacted.png
-   aksarc-workloads-redacted.png

------------------------------------------------------------------------

# Screenshot Inventory

  ------------------------------------------------------------------------------------------------------------------
  ID      Filename                                   Portal Location  Purpose         Documentation   Presentation
  ------- ------------------------------------------ ---------------- --------------- --------------- --------------
  SS-01   aksarc-cluster-overview-redacted.png       AKS Overview     Cluster         Yes             Yes
                                                                      deployment                      
                                                                      status                          

  SS-02   aksarc-nodepool-redacted.png               Node Pools       Worker node     Yes             Yes
                                                                      configuration                   

  SS-03   aksarc-workloads-redacted.png              Kubernetes       Workload        Yes             Yes
                                                     Resources →      validation                      
                                                     Workloads                                        

  SS-04   aksarc-service-loadbalancer-redacted.png   Services &       LoadBalancer    Yes             Yes
                                                     Ingresses        validation                      

  SS-05   aksarc-nginx-validation-redacted.png       Browser          End-to-end      Yes             Yes
                                                                      application                     
                                                                      validation                      

  SS-06   aksarc-extensions-redacted.png             Extensions       Arc Networking  Yes             Optional
                                                                      & Azure Monitor                 

  SS-07   aksarc-tags-redacted.png                   Tags             Governance      Yes             Optional
                                                                      validation                      

  SS-08   aksarc-monitoring-findings-redacted.png    Monitor          Monitoring      Yes             No
                                                                      findings                        

  SS-09   azurelocal-cluster-health-redacted.png     Azure Local      Platform health Yes             Optional
  ------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

# Screenshot Capture Guidelines

-   Capture immediately after successful validation.
-   Use browser zoom to improve readability.
-   Remove subscription IDs and tenant information.
-   Avoid browser bookmarks or personal profile information.
-   Crop unnecessary navigation panes when appropriate.

------------------------------------------------------------------------

# Documentation Mapping

  Screenshot   Referenced In
  ------------ --------------------------------------------
  SS-01        12_AKS_Enabled_by_Azure_Arc_Minimal_PoC.md
  SS-02        12_AKS_Enabled_by_Azure_Arc_Minimal_PoC.md
  SS-03        12_AKS_Enabled_by_Azure_Arc_Minimal_PoC.md
  SS-04        12_AKS_Enabled_by_Azure_Arc_Minimal_PoC.md
  SS-05        12_AKS_Enabled_by_Azure_Arc_Minimal_PoC.md
  SS-06        12.2_AKS_Evidence_Guide.md
  SS-07        12.2_AKS_Evidence_Guide.md
  SS-08        12.1_AKS_Troubleshooting.md
  SS-09        12.1_AKS_Troubleshooting.md

------------------------------------------------------------------------

# Presentation Usage

Mandatory:

-   SS-01
-   SS-02
-   SS-03
-   SS-04
-   SS-05

Optional:

-   SS-06
-   SS-09

Do Not Use:

-   Blank monitoring charts
-   Cloud Shell errors
-   Failed deployment screenshots

------------------------------------------------------------------------

# Final Review Checklist

-   Screenshot naming verified
-   Sanitization completed
-   Documentation references updated
-   Presentation mapping verified
-   Repository committed
