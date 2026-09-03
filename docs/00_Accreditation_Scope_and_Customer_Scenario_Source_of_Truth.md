# Accreditation Scope and Customer Scenario Source of Truth

## Purpose

This document is the authoritative source of truth for the Azure Local accreditation lab. It defines the governing customer scenario, the required accreditation activities, the implementation boundaries, and the evidence discipline.

All implementation decisions, lab work, documentation, diagrams, walkthroughs, and presentations must trace back to this document. If another repository document conflicts with this file, this file takes precedence until it is deliberately updated.

---

## 1. Governing Customer Scenario

### Customer

**Contoso Manufacturing**

Contoso Manufacturing is a global manufacturing organization operating across North America and Europe. It is evaluating Azure Local as part of a VMware exit strategy.

### Current environment

- Approximately 300 VMware virtual machines.
- Workloads include manufacturing systems, ERP, file services, Active Directory, SQL Server, and edge applications.
- Windows Server and Linux workloads are present.
- Three datacenters and twelve remote manufacturing sites are explicitly described in the available scenario information.
- The scenario also refers to 25 sites overall. The remaining locations are not sufficiently described and must remain an information gap rather than being invented.
- Existing Azure connectivity includes ExpressRoute.
- Operations and security are centrally managed.
- Backup and disaster recovery are required across sites.

### Business drivers

- Exit VMware within 18 months.
- Reduce virtualization licensing cost and platform complexity.
- Minimize migration downtime and preserve service levels.
- Retain on-premises control and data residency where required.
- Adopt a hybrid cloud operating model with Azure.
- Simplify operations and lifecycle management.
- Establish a platform that can support future edge and AI use cases.

### Technical requirements

- Support Windows and Linux virtual machines.
- Support SQL Server and line-of-business applications.
- Integrate with Microsoft Entra ID and Azure Arc.
- Provide centralized monitoring, governance, and management through Azure.
- Provide secure remote management.
- Support scalable growth.

### Resiliency requirements

- Cluster-level high availability.
- Site-level disaster recovery for critical workloads.
- Backup and recovery capabilities.
- Defined workload RPO and RTO targets.

### Security requirements

- BitLocker.
- Role-based access control and least privilege.
- Compliance controls.
- Security monitoring and auditability.

### Important information gap

The scenario states **25 sites**, while only **3 datacenters + 12 remote manufacturing sites = 15 locations** are explicitly described. The purpose, infrastructure, workload footprint, and requirements of the remaining locations are unknown. No architecture or sizing decision may silently invent those details.

---

## 2. Required Accreditation Activities

The following activities define the required scope. Work that does not support one of these activities should not take priority over the required tasks.

### Activity 1: Customer Discovery and Assessment

Required outcome:

- Assess the current environment, business drivers, technical requirements, risks, assumptions, dependencies, and suitability of Azure Local.
- Identify information gaps that require customer validation.

Required deliverable:

- A concise customer assessment, approximately 1 to 2 pages.

Current repository deliverable:

- `docs/01_Customer_Discovery_and_Assessment.md`

### Activity 2: Architecture Design

Required outcome:

Design an Azure Local solution covering:

- Compute
- Storage
- Networking
- Identity
- Azure Arc integration
- Security and governance
- Business continuity and disaster recovery
- Design rationale and assumptions

Required deliverables:

- Architecture diagram.
- Approximately 5 to 10 technical architecture slides.

Current repository deliverables include:

- `docs/02_Contoso_Azure_Local_Architecture.md`
- `diagrams/Contoso_Azure_Local_High_Level_Architecture.vdx`
- `diagrams/Contoso_Azure_Local_High_Level_Architecture_Editable.svg`
- `diagrams/contoso_azure_local_architecture_overview.png`

### Activity 3: Azure Local Proof of Concept using Jumpstart / LocalBox

This is the current implementation priority.

Required lab outcomes:

- Deploy or review the Azure Local cluster provided through the Jumpstart / LocalBox lab.
- Connect and validate Azure Local integration with Azure Arc.
- Create and manage at least one Azure Local virtual machine.
- Configure and validate logical workload networking.
- Demonstrate Azure-based monitoring and management.
- Demonstrate virtual machine lifecycle operations.
- Demonstrate Azure Local update and lifecycle operations.

Required deliverable:

- A lab walkthrough with evidence showing the required outcomes were actually performed or validated.

Implementation rule:

The lab uses the **Microsoft Azure Arc Jumpstart Azure Local / LocalBox approach** required for the accreditation exercise. The official LocalBox deployment mechanism may use Bicep automation. The accreditation objective is not to create a separate custom IaC solution. Deployment automation is only an enabling mechanism for the required lab outcomes above.

### Activity 4: VMware Migration Planning

Required outcome:

- Define the migration approach from VMware to Azure Local.
- Address migration tooling, workload dependencies, downtime, validation, rollback, migration waves, and key risks.

Required deliverable:

- VMware migration plan and strategy.

### Activity 5: Executive Presentation

Required outcome:

- Explain the customer problem, Azure Local recommendation, architecture, migration approach, operational model, risks, benefits, and next steps in an executive-friendly format.

Required deliverable:

- Approximately 10 to 15 slides.

### Activity 6: Customer Meeting Preparation

Required outcome:

- Prepare concise customer-facing talking points for discovery, design, migration, operational readiness, risks, and recommendations.

### Activity 7: Knowledge Validation

Required outcome:

- Complete the required written knowledge validation.

Required deliverable:

- Answers to 8 written validation questions.

---

## 3. Current Execution Priority

As of the operator-evidence checkpoint on **3 September 2026**, Activity 3 execution is complete within the recorded basic monitoring scope, including the solution update and agreed post-update validation. See [the final evidence record](11_Azure_Monitoring_and_Platform_Update_Validation.md).

**Current priority: review and finalize accreditation deliverables using the verified PoC evidence.**

This does not approve the production design or mark all accreditation activities complete. Review the presentation and VMware migration drafts, complete customer-meeting preparation, and obtain the eight written validation questions. Track governance expiry, guest activation and monitoring limitations separately; do not add optional infrastructure work without a scope reason.

Do not expand the lab into unrelated Azure Local features, custom infrastructure engineering, AKS, optional services, or additional IaC work unless a required accreditation outcome depends on it.

---

## 4. Implementation Decision Rule

Before any implementation step, ask:

1. Which required accreditation activity does this step support?
2. Which specific required outcome does it validate?
3. What evidence will prove completion?
4. Is the step required, or merely optional/interesting?

If the step is optional and does not materially support a required outcome, it should be deferred.

The execution sequence is:

**Source of Truth -> Required Task -> Implementation -> Verification -> Evidence -> Customer Explanation**

---

## 5. Evidence and Completion Discipline

- Planned work must never be presented as completed.
- A task is complete only after it has been executed or directly verified.
- Evidence should be captured for each required lab outcome.
- Public repository evidence must be sanitized before publication.
- Full subscription IDs, credentials, secrets, tokens, internal Microsoft content, confidential company information, and sensitive screenshots must not be committed.
- Where the customer scenario lacks information, record the item as an assumption, dependency, or information gap instead of inventing a value.

---

## 6. Repository Authority

This file is the **single point of truth for accreditation scope and the governing customer scenario**.

Supporting documents may contain deeper detail, analysis, commands, architecture rationale, or evidence, but they must remain consistent with this file.

The `main` branch remains the repository source of truth for verified work.
