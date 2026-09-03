# Azure Local JumpBox Accreditation Lab

This repository contains the hands-on Azure Local accreditation lab, customer assessment material, architecture design, VMware migration planning, operational evidence, and customer-facing deliverables for the Contoso Manufacturing scenario.

## Governing Source of Truth

The authoritative source for the **customer scenario, required accreditation tasks, implementation boundaries, execution priority, and evidence discipline** is:

`docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md`

All lab implementation, documentation, diagrams, walkthroughs, and presentations must remain traceable to that file. If another repository document conflicts with it, the `00` source-of-truth document takes precedence until it is deliberately updated.

## Purpose

The repository is intended to provide a clear and traceable record of the Azure Local accreditation activities, including:

1. Customer discovery and assessment
2. Azure Local architecture design
3. Azure Local Jumpstart / LocalBox proof of concept
4. VMware migration planning
5. Executive presentation content
6. Customer meeting preparation
7. Knowledge validation

## Governing Customer Scenario

The accreditation scenario is based on Contoso Manufacturing, a global manufacturing organization evaluating Azure Local as part of a VMware exit strategy. The design and lab decisions in this repository must be traceable back to the stated business, technical, resiliency, security, operational, and migration requirements.

The complete governing scenario and known information gaps are maintained only in `docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md`.

## Working Principle

Every major activity should follow this sequence:

**Source of Truth -> Required Task -> Implementation -> Verification -> Evidence -> Customer Explanation**

Work that does not materially support a required accreditation outcome should be deferred rather than expanding the lab scope.

## Lab Scope

The proof of concept will use the Microsoft Azure Arc Jumpstart Azure Local / LocalBox lab approach provided for the accreditation challenge. The lab must demonstrate, at minimum:

- Azure Local cluster deployment or review
- Azure Arc connectivity
- Azure Local VM creation and management
- Logical workload networking
- Azure-based monitoring and management
- VM lifecycle operations
- Azure Local update and lifecycle operations

The official LocalBox deployment automation is an enabling mechanism. The accreditation objective is to validate the required Azure Local outcomes, not to create a separate custom IaC solution.

## Public Repository Security Rules

This is a public repository. The following content must never be committed:

- Passwords, tokens, secrets, certificates, private keys, or connection strings
- Terraform state or any other file containing secrets or environment-specific credentials
- Sensitive tenant, subscription, account, or user information unless explicitly sanitized
- Screenshots that expose credentials, access tokens, personally identifiable information, or confidential customer information
- Microsoft Confidential, NDA-restricted, or internal support-only content
- Internal Microsoft support wiki content or restricted troubleshooting material

All screenshots, logs, scripts, and evidence must be reviewed and sanitized before publication.

## Source of Truth

The `main` branch of this repository is the source of truth for verified work. Within the repository, `docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md` is the governing source for accreditation scope and customer scenario.

Work should be documented only after it has been executed or verified. Planned work must be clearly identified as planned and must not be represented as completed.

## Status

Customer discovery, architecture design, and Azure subscription readiness have been performed. The current implementation priority is the required Jumpstart / LocalBox proof of concept defined in Activity 3 of the governing source-of-truth document.

## Accreditation Delivery Status

Status reviewed on **3 September 2026** against [the governing scope](docs/00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md), the repository inventory, prior session decisions, and published validation records. An artifact not found here is not assumed to be complete elsewhere.

| Activity | Available evidence or content | Remaining deliverable / status |
| --- | --- | --- |
| 1. Discovery | [Assessment and discovery questionnaire](docs/01_Customer_Discovery_and_Assessment.md) | Content available; final concise 1–2-page submission format has not been verified |
| 2. Architecture | [Architecture baseline](docs/02_Contoso_Azure_Local_Architecture.md) and diagram files under `diagrams/` | Baseline and diagrams available; required 5–10 technical slides not found or confirmed complete |
| 3. LocalBox PoC | [Walkthrough](docs/04_Azure_LocalBox_Lab_Walkthrough.md), [VM/storage evidence](docs/08_Azure_Local_VM_WAC_Storage_and_Disk_Validation.md), [VM lifecycle evidence](docs/09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md), and [recovery checkpoint](docs/10_Lab_Recovery_Checkpoint_and_Follow_Up.md) | IN PROGRESS: monitoring/management NEXT; Azure Local platform update/lifecycle PENDING |
| 4. VMware migration | High-level migration considerations in the architecture and production-concepts documents | Dedicated strategy covering tooling, dependencies, waves, downtime, validation and rollback not found; PENDING |
| 5. Executive presentation | Customer and technical source material available | Required 10–15-slide executive deck not found; PENDING |
| 6. Customer meeting preparation | Discovery questions and architecture rationale available as inputs | Dedicated meeting talking points / preparation deliverable not found; PENDING |
| 7. Knowledge validation | Scope specifies eight written questions | Exact question set and completed answers not found in this repo; PENDING |

### Scope and evidence boundaries

- Completed Activity 3 checks include deployment, Arc connectivity, logical networking, VM creation, NIC/IP, guest management, WAC/storage validation, and workload VM stop/start/restart. Monitoring and Azure Local platform lifecycle are separate outcomes.
- The portal screenshot showing an Up to date state is a recorded status snapshot, not evidence that a platform update operation was performed.
- Architecture diagrams represent a proposed customer design, not the deployed LocalBox topology or proof that every pictured service was implemented. Review service labels, connectivity assumptions and support mappings before final slide delivery.
- The customer site's count discrepancy remains open: 25 stated, 15 described, 10 undefined. Workload sizing, site placement, network characteristics and approved RPO/RTO are also discovery inputs, not confirmed facts.
- OS-disk expansion guidance was documented; no resize was executed. Optional production topics in `docs/07` do not add mandatory lab tasks.
- Permanent governance resolution and laptop repository synchronization remain operational follow-ups, separate from completed cluster recovery.
- The immediate task remains Activity 3 monitoring prerequisite verification. This audit does not authorize skipping to later activities or claim they were executed.

### Documentation completion check

After each verified outcome, update its evidence record, affected dependency/runbook entries, current-status tables, this delivery matrix, and next-step pointers together. Verify semantic consistency and links as well as reading back the published files. Keep historical snapshots identifiable and distinguish completed tests from fresh live state.

## Operational Checkpoint: 3 September 2026

Operator-supplied checks confirm both cluster nodes Up, Cloud Witness and Cluster Group Online, and all three cluster shared volumes Online.

A governance follow-up reminder is scheduled for **14 September 2026 around 09:00 IST**, before the temporary lab waivers expire at **15 September 2026, 00:00 IST**. Permanent governance resolution remains pending.

See [Lab Recovery Checkpoint and Follow-up](docs/10_Lab_Recovery_Checkpoint_and_Follow_Up.md) for the reminder, expiry and next task. Activity 3 monitoring prerequisite verification is next.
