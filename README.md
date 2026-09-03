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

## Operational Checkpoint: 3 September 2026

Operator-supplied checks confirm both cluster nodes Up, Cloud Witness and Cluster Group Online, and all three cluster shared volumes Online.

A governance follow-up reminder is scheduled for **14 September 2026 around 09:00 IST**, before the temporary lab waivers expire at **15 September 2026, 00:00 IST**. Permanent governance resolution remains pending.

See [Lab Recovery Checkpoint and Follow-up](docs/10_Lab_Recovery_Checkpoint_and_Follow_Up.md) for the reminder, expiry and next task. Activity 3 monitoring prerequisite verification is next.
