# Customer Discovery and Assessment

## Contoso Manufacturing

### 1. Engagement Context

Contoso Manufacturing is evaluating alternatives to its current VMware platform as it approaches a licensing renewal. The company operates across North America and Europe and currently supports approximately 300 virtual machines across datacenters and manufacturing sites. These workloads include manufacturing systems, ERP, file services, Active Directory, SQL Server, and edge applications.

The main business objective is to exit VMware within 18 months while maintaining operational stability. Contoso also wants to reduce virtualization licensing cost, retain on-premises control where required, improve hybrid cloud integration with Azure, simplify day-to-day operations, and create a platform that can support future edge and AI use cases.

Azure Local appears to be a strong candidate because it can provide an Azure-connected operating model while keeping workloads on customer-controlled infrastructure. A final recommendation, however, should only be made after validating workload suitability, site readiness, networking, storage, resiliency, migration dependencies, and operational capabilities.

## 2. Current Environment Summary

The information provided so far indicates the following baseline:

- Approximately 300 VMware virtual machines.
- Three datacenters and twelve remote manufacturing sites are explicitly described.
- Windows Server and Linux workloads are both present.
- SQL Server supports business-critical applications.
- Active Directory is integrated across the environment.
- ExpressRoute connectivity to Azure already exists.
- Operations and security functions are centrally managed.
- Backup and disaster recovery are required across sites.
- Contoso wants to adopt a hybrid cloud operating model with Azure Arc and Microsoft Entra integration.

One point requires clarification during discovery. The customer scenario refers to 25 sites overall, while the current environment description explicitly identifies three datacenters and twelve remote manufacturing sites. The role, workload footprint, and infrastructure of the remaining locations need to be confirmed before a final site architecture is proposed.

## 3. Business and Technical Drivers

### Business drivers

- Exit VMware before the next major renewal cycle, with an 18-month target.
- Reduce ongoing virtualization licensing and platform complexity.
- Protect manufacturing availability and existing service-level commitments during migration.
- Retain on-premises control and data residency where business or regulatory requirements demand it.
- Move toward a consistent hybrid cloud operating model rather than managing isolated infrastructure platforms.
- Establish a foundation for future edge, analytics, and AI workloads.

### Technical drivers

- Support existing Windows and Linux virtual machines.
- Continue hosting SQL Server and line-of-business applications.
- Integrate on-premises infrastructure with Azure Arc and Microsoft Entra ID.
- Centralize monitoring, governance, policy, and security visibility through Azure.
- Provide secure remote administration for distributed sites.
- Deliver cluster-level high availability and site-level disaster recovery for critical workloads.
- Support BitLocker, role-based access control, least privilege, security monitoring, and auditability.
- Provide a scalable platform that can grow as manufacturing and edge requirements change.

## 4. Initial Suitability Assessment

Azure Local is aligned with many of Contoso's stated requirements. It is particularly relevant where workloads must remain on-premises but the customer still wants Azure-based governance, Arc-enabled management, integrated monitoring, policy, and lifecycle operations.

The existing Azure connectivity is also an advantage because Contoso already has ExpressRoute and centralized cloud operations. This may reduce the organizational change required to adopt an Azure-connected operating model.

The main concern is not whether Azure Local can run virtual machines. The more important questions are whether each site can meet supported infrastructure requirements, whether business-critical applications have clearly defined recovery objectives, whether the network can support Azure-connected operations, and whether VMware workloads can be grouped into realistic migration waves without creating unacceptable business disruption.

A proof of concept should therefore validate the operating model, not only VM creation. The accreditation lab will focus on cluster review, Azure Arc integration, VM lifecycle management, logical networking, Azure-based monitoring, and update operations.

## 5. Key Risks, Assumptions and Dependencies

| Area | Initial observation |
| --- | --- |
| Site inventory | The scenario states 25 sites, but only 15 locations are explicitly described. Site purpose and infrastructure must be reconciled. |
| VMware estate | VM count is known, but CPU, memory, storage, IOPS, latency sensitivity, dependencies, and growth data are not yet available. |
| SQL Server | Business criticality is known, but HA topology, versions, licensing model, database size, RPO, RTO, and application dependencies are unknown. |
| Network | ExpressRoute exists, but site-by-site WAN bandwidth, latency, internet breakout, firewall rules, VLAN design, and Azure connectivity paths require validation. |
| Hardware | No supported Azure Local hardware inventory, node sizing, storage topology, or lifecycle status has been provided. |
| Resiliency | HA and DR are required, but workload tiers and agreed RPO/RTO values have not yet been defined. |
| Migration | The 18-month target appears achievable only with structured workload discovery, dependency mapping, migration waves, testing, and business owner sign-off. |
| Operations | Central teams exist, but current VMware operational processes, monitoring tools, backup tools, patching practices, and support responsibilities need assessment. |
| Security | Required controls are known at a high level, but regulatory frameworks, logging retention, privileged access model, segmentation, and security ownership need clarification. |
| Public cloud dependency | Azure Local is Azure-connected. Connectivity, identity, DNS, proxy, firewall, and outbound endpoint requirements must be validated for each deployment pattern. |

## 6. Discovery Questionnaire

The following questions should be used during the initial customer workshops.

### Business and scope

1. Which sites are included in the VMware exit program, and what is the role of each location?
2. What happens if the 18-month migration deadline is missed?
3. Which workloads are considered business critical, manufacturing critical, or non-critical?
4. Are there regulatory or contractual requirements that restrict where specific workloads or data can run?
5. What service-level commitments must remain unchanged during and after migration?

### Workloads and applications

6. Can Contoso provide a complete VMware inventory including CPU, memory, storage, operating system, utilization, and growth history?
7. Which VMs have application, database, network, licensing, or hardware dependencies?
8. Which applications require low latency to manufacturing equipment or local plant systems?
9. Which Windows and Linux versions are currently in use?
10. Which workloads are candidates for retirement, consolidation, modernization, or direct migration?

### SQL Server

11. Which SQL Server versions, editions, and licensing models are in use?
12. Which databases require high availability or disaster recovery?
13. What are the database sizes, transaction rates, storage performance requirements, RPOs, and RTOs?
14. Are SQL Server Always On, failover clustering, replication, or third-party protection technologies already used?

### Infrastructure and sites

15. What server, storage, and network hardware exists at each datacenter and manufacturing site?
16. Which sites have sufficient space, power, cooling, and physical support capability for Azure Local systems?
17. Are there standard hardware procurement or vendor requirements?
18. What is the expected infrastructure growth over the next three to five years?

### Networking and connectivity

19. What are the WAN bandwidth and latency characteristics for every site?
20. Which sites use ExpressRoute, VPN, internet breakout, or centralized egress?
21. What VLANs, IP ranges, DNS services, proxies, and firewall policies are currently in place?
22. Are manufacturing networks segmented from corporate and management networks?
23. Is there a requirement for disconnected or highly constrained operations at any location?

### Identity, security and governance

24. How is Active Directory designed across sites, and where are domain controllers located?
25. What Microsoft Entra ID and Azure Arc capabilities are already in use?
26. What privileged access and administrative role model is currently followed?
27. Which compliance standards apply to Contoso and its manufacturing operations?
28. What are the required logging, monitoring, security incident, and audit retention periods?
29. Are BitLocker, Defender, Azure Policy, or other security baselines already mandated?

### Availability, backup and disaster recovery

30. What are the agreed RPO and RTO targets for each workload tier?
31. Which applications require site-level disaster recovery?
32. What backup products, retention policies, immutability controls, and restore testing processes are currently used?
33. Which sites can act as recovery locations for one another?
34. How frequently are disaster recovery procedures tested today?

### Operations and lifecycle

35. Who owns virtualization, operating system, networking, storage, security, and Azure operations today?
36. What monitoring, ITSM, patching, configuration management, and automation tools are currently used?
37. What maintenance windows are available at manufacturing sites?
38. How are infrastructure firmware, driver, operating system, and platform updates currently governed?
39. What support model is expected after Azure Local adoption?

### Migration

40. Is Azure Migrate already used anywhere in the organization?
41. Can migration appliances be deployed in the source VMware environments and target Azure Local environments?
42. What application outage windows are acceptable for migration?
43. Which workloads require migration rehearsals or business validation before cutover?
44. Are there applications that cannot be migrated until vendor certification is confirmed?
45. How will rollback decisions be made during migration waves?

## 7. Recommended Next Step

The next engagement step should be a structured discovery workshop followed by workload and site readiness assessment. The collected data should then be used to classify workloads by criticality, migration approach, recovery requirement, and target site.

Only after that assessment should the final Azure Local topology, node sizing, network design, storage design, and disaster recovery model be locked. The accreditation architecture exercise can use reasonable assumptions where the scenario does not provide enough detail, but those assumptions must remain clearly identified rather than presented as confirmed customer facts.
