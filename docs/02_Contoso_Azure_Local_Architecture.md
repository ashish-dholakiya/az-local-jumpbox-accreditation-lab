# Contoso Manufacturing Azure Local Architecture Baseline

## Purpose

This document defines the initial high-level Azure Local architecture for the Contoso Manufacturing scenario. It is intended to support the accreditation architecture exercise and provide a technically defensible baseline for the architecture diagram and design rationale presentation.

This is not a final production design. Several customer inputs are still missing, including workload sizing, detailed application dependencies, site-by-site capacity, network characteristics, and approved RPO/RTO values. Those items are captured as open decisions rather than assumed.

## 1. Customer Context

Contoso Manufacturing is planning to exit VMware within 18 months while maintaining on-premises control, data residency, operational continuity, and integration with Azure services. The current estate includes approximately 300 virtual machines across datacenters and manufacturing sites, with Windows Server, Linux, Active Directory, SQL Server, ERP, file services, and edge applications.

Azure Local is a strong candidate because it can provide a consistent Azure operating model on customer-owned infrastructure while continuing to run traditional virtualized workloads close to the sites where they are needed.

## 2. High-Level Architecture Direction

The proposed direction is a distributed Azure Local platform with appropriately sized clusters at selected datacenter and manufacturing locations. Each cluster would be connected to Azure through Azure Arc for centralized governance, monitoring, security, inventory, policy, and lifecycle operations.

The exact number of clusters and the node count per site must be determined after workload discovery and capacity analysis. Datacenter locations are expected to use resilient multi-node clusters, while smaller manufacturing sites may use smaller validated Azure Local configurations where justified by workload demand and availability requirements.

## 3. Compute

### Proposed Design

- Use Microsoft-validated Azure Local hardware from an approved OEM partner.
- Size clusters according to CPU, memory, storage, growth, and failure-domain requirements rather than simply matching the existing VMware host count.
- Use multi-node clusters for business-critical datacenter workloads.
- Place Windows and Linux virtual machines on Azure Local using Azure Local VM management.
- Maintain sufficient spare capacity to support planned maintenance and node failure scenarios.
- Separate critical SQL Server and ERP capacity planning from general-purpose VM consolidation because their performance and availability requirements may differ significantly.

### Rationale

The goal is to replace the virtualization platform without recreating the existing VMware topology blindly. Capacity should be based on measured utilization, growth, resilience targets, and workload criticality. This reduces the risk of carrying old overprovisioning or hidden constraints into the new platform.

## 4. Storage

### Proposed Design

For standard hyperconverged deployments, use Storage Spaces Direct with validated NVMe or SSD-based storage. For latency-sensitive workloads, all-flash configurations should be strongly considered.

Storage design must account for:

- VM capacity and growth
- SQL Server IOPS and latency requirements
- Resiliency overhead
- Backup retention
- Rebuild time after drive or node failure
- Available capacity during maintenance or degraded operation

Where Contoso already has a strategic SAN platform or requires independent compute and storage scaling, Azure Local external storage options can be assessed separately. This should be an evidence-based exception rather than the default assumption.

### Rationale

Storage is one of the highest-risk areas in a virtualization migration. A design that only compares raw terabytes can fail even when capacity appears sufficient. Workload latency, IOPS, write patterns, resiliency, and rebuild behaviour must be considered.

## 5. Networking

### Physical Network

Datacenter clusters should use redundant physical switching with no single point of failure. A typical resilient design uses dual top-of-rack switches and redundant network paths from each Azure Local node.

Network roles should be designed around management, compute/workload, and storage traffic. Network ATC should be used where appropriate to apply supported network intents and reduce configuration drift.

### Workload Networking

Use Azure Local logical networks to represent workload VLANs and IP ranges. Segmentation should reflect business and security boundaries rather than placing all migrated VMs into a single network.

Typical logical separation may include:

- Management
- Production application workloads
- Database workloads
- Manufacturing or OT-adjacent workloads
- Shared services
- Backup or replication traffic where required

The final VLAN and address plan must align with Contoso's existing network architecture.

### WAN and Azure Connectivity

Contoso already has ExpressRoute connectivity to Azure. The design should evaluate how existing ExpressRoute connectivity can support centralized Azure services and operational access without assuming that every remote site has identical WAN capability.

Internet or Azure service connectivity requirements for Azure Local must also be validated for each site, including proxy, firewall, DNS, and outbound endpoint requirements.

### Rationale

Manufacturing environments are distributed and often have different site connectivity characteristics. Networking therefore needs to be designed per site class rather than using one universal pattern.

## 6. Identity and Access

### Proposed Design

- Continue using Active Directory for workloads that depend on traditional domain services.
- Integrate operational access with Microsoft Entra ID and Azure role-based access control where supported.
- Use Azure Arc to bring Azure Resource Manager based control and governance to the Azure Local environment.
- Apply least privilege through role-based access rather than shared administrative accounts.
- Separate platform administration, security operations, and workload administration responsibilities.

### Rationale

The customer already has Active Directory integrated workloads, so the architecture should not force an unnecessary identity redesign during the VMware exit. The migration can preserve workload compatibility while introducing stronger Azure-based governance for platform operations.

## 7. Azure Arc Integration

Azure Arc is central to the proposed operating model. Each Azure Local environment should be connected to Azure and represented through Azure Resource Manager.

Azure Arc enables Contoso to use Azure as the centralized management and governance plane across distributed locations. The target operating model should include, where applicable:

- Azure Policy
- Azure Monitor
- Microsoft Defender for Cloud
- Resource inventory and tagging
- Azure RBAC
- Update and lifecycle management
- Azure Local VM management
- Arc-enabled workload services where justified

### Rationale

The value of Azure Local is not limited to replacing the hypervisor. Azure Arc gives the customer a common management model across Azure and on-premises infrastructure, which directly supports the requirement to reduce operational complexity.

## 8. Security and Governance

### Platform Security

The design should include:

- BitLocker for data-at-rest protection where applicable
- Secure Boot and TPM-backed platform controls supported by validated hardware
- Role-based access control and least privilege
- Centralized logging and security monitoring
- Azure Policy for governance and compliance enforcement
- Microsoft Defender for Cloud for security posture visibility where selected
- Administrative access through controlled management paths
- MFA and modern identity controls for privileged Azure operations

### Governance

Contoso's centralized operations and security teams should define standard policies for naming, tagging, permitted regions, resource ownership, security posture, monitoring, and exception handling.

### Rationale

A distributed manufacturing estate becomes difficult to manage if every site develops its own operating model. Central policy and monitoring reduce drift while still allowing workloads and data to remain on-premises.

## 9. Business Continuity and Disaster Recovery

### Cluster-Level High Availability

Use Failover Clustering and Storage Spaces Direct resiliency to protect against supported node, disk, and network failures within a site. Critical clusters should be sized so that business workloads can continue to operate during expected failure and maintenance scenarios.

### Site-Level Disaster Recovery

Cluster high availability does not protect against complete site failure. Critical workloads therefore require a separate DR strategy.

The final solution may combine:

- Azure Site Recovery or supported replication mechanisms for appropriate VMs
- Application-native replication for SQL Server or other business-critical platforms
- Azure Backup or supported enterprise backup platforms
- Secondary Azure Local sites where justified
- Azure-based recovery targets where business and regulatory requirements permit

The technology must be selected after Contoso confirms application-specific RPO and RTO targets.

### Backup

Backup should remain independent of high availability and replication. The design should define recovery points, retention, immutability requirements, off-site copies, and regular restore testing.

### Rationale

High availability, backup, and disaster recovery solve different failure scenarios. Treating them as interchangeable would leave significant recovery gaps.

## 10. Operations and Lifecycle

The target operating model should centralize routine platform operations where possible while maintaining local resiliency.

Operational responsibilities should include:

- Health monitoring
- Capacity management
- VM lifecycle management
- Firmware and driver lifecycle
- Azure Local solution updates
- Security patching
- Backup verification
- DR testing
- Certificate and identity lifecycle
- Configuration compliance

A maintenance process should define change windows, pre-checks, rollback planning, validation, and evidence capture.

## 11. Site Strategy

The scenario states that Contoso has 25 sites across North America and Europe, while the existing-environment description explicitly identifies 3 datacenters and 12 remote manufacturing sites. This accounts for 15 sites, leaving 10 sites undefined.

No architecture decision should silently infer the role or infrastructure requirements of those additional locations.

The discovery phase must confirm:

- Which sites currently host VMware workloads
- Which sites require local compute after migration
- Which locations can consolidate workloads into regional datacenters
- Which sites need low-latency local processing
- Which sites have sufficient network connectivity for centralized services
- Which sites have regulatory or data residency constraints

This information will determine the final cluster placement strategy.

## 12. Initial Site Pattern

The following pattern is proposed only as a design framework until discovery is complete.

### Tier 1: Primary Datacenter Sites

Likely characteristics:

- Larger multi-node Azure Local clusters
- Business-critical SQL Server and ERP workloads
- Dual switching and high-capacity networking
- Strong local HA
- Site-level DR to another datacenter or approved Azure target

### Tier 2: Manufacturing Sites with Local Compute Requirements

Likely characteristics:

- Smaller validated Azure Local clusters
- Local manufacturing and edge workloads
- Local survivability where WAN disruption is a concern
- Centralized Azure Arc governance
- DR based on workload criticality

### Tier 3: Sites Without Strong Local Compute Requirements

These locations may not require an Azure Local deployment. Workloads could potentially be consolidated into another Azure Local site or Azure, subject to latency, operational, regulatory, and business requirements.

## 13. Major Design Decisions and Status

| Design Area | Initial Direction | Status |
|---|---|---|
| Platform | Azure Local for appropriate on-premises and edge virtualization workloads | Proposed |
| Management plane | Azure Arc and Azure Resource Manager | Proposed |
| Datacenter cluster pattern | Resilient multi-node Azure Local cluster | Proposed |
| Remote site pattern | Size and topology based on workload and site criticality | Discovery required |
| Storage | Storage Spaces Direct, preferably flash for performance-sensitive workloads | Proposed |
| Networking | Redundant physical network plus segmented logical workload networks | Proposed |
| Identity | Existing AD for workload compatibility, Entra ID and Azure RBAC for Azure management | Proposed |
| Security | BitLocker, RBAC, Policy, centralized monitoring and security controls | Proposed |
| Backup | Independent backup strategy with restore testing | Required |
| Site DR | Workload-specific strategy based on approved RPO/RTO | Discovery required |
| ExpressRoute | Reuse existing connectivity where technically appropriate | Validation required |
| VMware migration | Phased migration using supported tooling and dependency mapping | Separate migration workstream |

## 14. Information Required Before Final Production Design

The following items must be confirmed before hardware sizing or final site architecture is approved:

1. VM CPU, memory, storage, and utilization data.
2. Application dependency mapping.
3. SQL Server topology, editions, HA model, and performance requirements.
4. Per-site workload inventory.
5. Clarification of the 25-site versus 15-described-site difference.
6. Network bandwidth, latency, packet loss, and redundancy by site.
7. Current VLAN, IP, routing, firewall, DNS, proxy, and NTP design.
8. Required RPO and RTO by application tier.
9. Backup retention and compliance requirements.
10. Data residency constraints by country or workload.
11. Existing hardware refresh dates and datacenter constraints.
12. Capacity growth forecast for at least three to five years.
13. Planned AI, edge, or container workloads.
14. Preferred OEM and support model.
15. Operational ownership and escalation model.

## 15. Accreditation Position

This architecture intentionally separates confirmed customer requirements from proposed design choices. The recommended approach is to use Azure Local as a distributed virtualization and hybrid infrastructure platform, with Azure Arc providing a common Azure-based operational model.

The architecture should be finalized only after discovery validates workload placement, capacity, networking, recovery objectives, and site requirements. This approach supports Contoso's VMware exit while avoiding a simple one-for-one infrastructure replacement that could carry existing inefficiencies into the new platform.
