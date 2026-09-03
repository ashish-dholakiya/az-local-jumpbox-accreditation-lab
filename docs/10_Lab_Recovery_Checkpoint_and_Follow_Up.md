# Lab Follow-up and Recovery Checkpoint

**Recorded: 3 September 2026**

## Governance follow-up reminder

A one-time ChatGPT reminder was created and enabled for **14 September 2026, around 09:00 IST**, to follow up with the subscription governance team before the temporary lab waivers expire.

| Item | Date and time |
| --- | --- |
| Reminder | 14 September 2026, around 09:00 IST |
| Waiver expiry | 15 September 2026, 00:00 IST |
| Equivalent expiry in UTC | 14 September 2026, 18:30 UTC |

The expiry is at the start of 15 September in India. The reminder uses a flexible schedule and runs within approximately one hour of the scheduled time. It does not extend exemptions or make Azure configuration changes.

The lab operator should coordinate the resolution or an approved extension with subscription governance before expiry. Permanent governance resolution remains pending. Detailed policy assignments, exemption identifiers, affected resource names and raw diagnostic evidence are intentionally omitted from this public note.

## Recovery checkpoint

Operator-supplied PowerShell output confirmed:

- Both cluster nodes: Up.
- Cloud Witness: configured and Online.
- Cluster Group: Online.
- All three cluster shared volumes: Online.

This records the observed recovery checkpoint, not a guarantee of continued health across future restarts or governance evaluations.

## Next task

Resume Activity 3 monitoring prerequisite verification, starting with the `AzureEdgeTelemetryAndDiagnostics` extension. Azure-based monitoring evidence and Azure Local platform update/lifecycle validation remain pending.

Before local repository work, verify the folder, branch and working-tree state, then synchronize main according to the multi-device SOP. This remote documentation update does not synchronize the operator's laptop.

See [the preceding validation checkpoint](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) and [the governing accreditation scope](00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md).
