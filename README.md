# Azure Local JumpBox Accreditation Lab

This repository contains the hands-on Azure Local accreditation lab, customer assessment material, architecture design, VMware migration planning, operational evidence, and customer-facing deliverables for the Contoso Manufacturing scenario.

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

## Working Principle

Every major activity should follow this sequence:

**Discover -> Assess -> Design -> Validate -> Evidence -> Explain**

Where practical, technical decisions should map to:

**Customer Requirement -> Azure Local Capability -> Design Decision -> Lab Validation -> Evidence -> Customer Talking Point**

## Lab Scope

The proof of concept will use the Microsoft Azure Arc Jumpstart Azure Local / LocalBox lab approach provided for the accreditation challenge. The lab must demonstrate, at minimum:

- Azure Local cluster deployment or review
- Azure Arc connectivity
- Azure Local VM creation and management
- Logical workload networking
- Azure-based monitoring and management
- VM lifecycle operations
- Azure Local update and lifecycle operations

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

The `main` branch of this repository is the source of truth for the accreditation lab and its deliverables. Work should be documented only after it has been executed or verified. Planned work must be clearly identified as planned and must not be represented as completed.

## Status

Repository initialized. Accreditation execution is starting with customer discovery and assessment, followed by architecture design and the Jumpstart / LocalBox proof of concept.
