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

## Subsequent monitoring and update evidence

[Monitoring and platform update validation](../../docs/11_Azure_Monitoring_and_Platform_Update_Validation.md) is a sanitized text record of operator-supplied portal screenshots and PowerShell results reviewed on 3 September 2026. It records basic metrics verification, healthy readiness, preparation, successful installation of 12.2608.1003.9 and fresh post-update validation, including interactive guest login.

The corresponding raw screenshots have not been published here. The older Up to date image remains historical and must not be used as evidence that version 12.2608.1003.9 was installed. Final installation and post-update evidence are now recorded in docs/11 as sanitized text. The guest activation watermark and missing network metric values remain limitations, not silently closed items.

