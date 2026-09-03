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


## Issue record: Cloud Witness access failure after startup

### Observed evidence and limits

| Observation | Conclusion supported by the session |
| --- | --- |
| Nodes and volumes initially showed transitional or failed states, then recovered | Verify actual backend state before attempting repair |
| Cloud Witness remained Failed; event 1659 reported Azure Storage access denied | Witness access remained a separate unresolved issue after volume recovery |
| Activity Log showed a successful Modify action on allowSharedKeyAccess and a write response with false | A governance-driven shared-key access change was confirmed |
| A node-side request returned HTTP 403 AuthorizationFailure; request and Azure response UTC times matched | The service responded; this check did not show clock skew or establish an invalid key |
| Public network access was Disabled and no private endpoints were shown | The intended storage network path was unavailable |
| Network settings were reported to disappear after a change | The exact policy or automation responsible was not established |
| A later check showed no witness and empty QuorumResource, with Cluster Group Online | Witness configuration was absent at that point; the cause of its removal was not established |

Volumes recovered before the final witness repair. Do not attribute their initial failures to Cloud Witness without additional evidence.

### Recovery performed

The operator restored permitted storage authentication and the intended selected-network access under a temporary governance workaround. Settings were rechecked in the portal. Failed witness-start attempts were followed by diagnosis rather than treated as successful recovery.

Once the missing witness configuration was confirmed, the operator ran Set-ClusterQuorum locally on a cluster node with the existing account key. The command succeeded, QuorumResource became Cloud Witness, and the witness reported Online. Final node, Cluster Group and CSV checks confirmed the recovery state listed above. No key regeneration or storage rebuild was required.

This is a sanitized account of operator-supplied evidence. Internal resource/assignment names, exemption identifiers and raw logs remain excluded. Permanent governance resolution, attribution of the network reversals, and stability across future restarts remain open.

### Related operational documentation

- [Dependency register](05_Azure_Local_Accreditation_Dependency_Register.md): ongoing witness, authentication, network and governance dependencies.
- [Deployment runbook, section 21.1](06_Azure_LocalBox_Deployment_Runbook.md#211-resume-an-existing-lab-and-troubleshoot-cloud-witness): read-only checks, conditional recovery and closure criteria.
- [Production concepts](07_Azure_Local_Production_Concepts_and_Further_Understanding.md): ownership and governance lessons, separated from lab evidence.

## Next task

The subsequent [monitoring and update evidence](11_Azure_Monitoring_and_Platform_Update_Validation.md) records successful installation of 12.2608.1003.9 and fresh post-update health, witness/quorum, VM login and basic monitoring evidence. The earlier recovery record remains historical; the later checks independently establish the final checkpoint.

Next: review final accreditation deliverables and presentation drafts. Guest Windows activation and network-metric coverage remain follow-ups; the governance reminder and waiver expiry are unchanged. A separate forgotten guest-password issue was resolved through authorized Azure Arc Run Command using protected input, followed by successful interactive login. No VM rebuild or cluster-password change was required.

Before local repository work, verify the folder, branch and working-tree state, then synchronize main according to the multi-device SOP. This remote documentation update does not synchronize the operator's laptop.

See [the preceding validation checkpoint](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) and [the governing accreditation scope](00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md).

