# Portal Evidence

This folder stores sanitized Azure Portal screenshots used as supporting evidence for the Azure Local accreditation lab.

## Rules

- Do not upload screenshots that expose full subscription IDs, tenant IDs, account identifiers, passwords, tokens, or other sensitive values.
- Prefer sanitized screenshots that preserve only the technical evidence required for the accreditation record.
- Use clear, descriptive filenames.

## Verified evidence

- `azure-local-cluster-overview-redacted.png`
  - Sanitized Azure Local cluster overview.
  - Supports verification of the deployed `localboxcluster` portal management experience, machine count, region, and connected state.

- `azure-local-cluster-overview-updates.png`
  - Sanitized Azure Local updates and management view.
  - Supports verification that the cluster was accessible in Azure Portal and showed an `Up to date` update state at the time of capture.

These screenshots are referenced from `docs/04_Azure_LocalBox_Lab_Walkthrough.md` under the Portal Validation Evidence section.
