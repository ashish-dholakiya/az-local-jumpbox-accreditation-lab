# Azure Monitoring and Platform Update Validation

**Evidence checkpoint: 3 September 2026. Installation remains IN PROGRESS.**

## 1. Purpose and evidence boundary

This record continues [VM lifecycle validation](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) and the [recovery checkpoint](10_Lab_Recovery_Checkpoint_and_Follow_Up.md) for Activity 3 of the [governing accreditation scope](00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md).

Evidence consists of operator-supplied Azure Portal screenshots and node-side PowerShell output reviewed during the session. This is a sanitized text record, not a claim of direct access to the running cluster. Raw screenshots, tenant/subscription identifiers, account details and operation identifiers have not been published.

All results are point-in-time observations. Pre-update health and earlier VM lifecycle PASS results are not post-update validation. The nested Australia East LocalBox environment is not a production performance benchmark.

## 2. Azure monitoring verification

| Check | Recorded evidence | Result |
| --- | --- | --- |
| AzLHOST1 telemetry extension | TelemetryAndDiagnostics 2.0.48.0; Succeeded; automatic upgrade Enabled | Provisioning PASS |
| AzLHOST2 telemetry extension | TelemetryAndDiagnostics 2.0.48.0; Succeeded; automatic upgrade Enabled | Provisioning PASS |
| Azure-hosted CPU and memory graphs | Recent last-hour data displayed; approximately 9.27% average CPU and 16.5 GB average memory for that selected window | Basic metrics viewing PASS |
| Azure-hosted disk graphs | Read/write operation and byte-rate graphs contained data in the 24-hour view | Basic metrics viewing PASS |
| Network metrics | Legends displayed missing values (`--`) | Not verified; do not interpret as confirmed zero traffic |
| Azure Local registration | Clustered, Registered, Connected in Get-AzureStackHCI output | Connection snapshot verified |
| Recent heartbeat alerts | No heartbeat alerts listed in the later last-hour portal view | No recent recurrence visible in that window; not permanent closure |

**Outcome:** basic Azure-based monitoring demonstration is verified, alongside the previously completed Azure management and workload lifecycle evidence. This does not establish comprehensive monitoring coverage, alert delivery testing, Log Analytics/Insights configuration or a production monitoring design.

### Portal session and heartbeat observations

Initially, metric tiles displayed Session expired and the alerts pane failed to load. A later portal view loaded alerts and CPU, memory and disk data. No agent reinstall or cluster repair was performed for that display symptom.

The 24-hour view showed 32 fired alerts; visible entries were titled Lost heartbeat from your cluster. The inspected instance had severity Error, alert condition Fired and user response New. These fields alone do not prove that all historical instances represent current failures.

The inspected alert fired at 09:43 IST on 3 September. The node's LastConnected value, interpreted using its displayed UTC-07:00 offset, corresponded to 10:30 IST that day. Later, the last-hour portal view showed no heartbeat alerts and recent CPU/memory samples. The underlying heartbeat interruption cause and resolution status of every historical alert were not established. Alerts were not manually closed or disabled.

## 3. System health recovery before preparation

The portal initially reported Critical update readiness and seven errors:

- Storage services, cluster shared volume, volume and virtual-disk health checks.
- A detached virtual-disk fault.
- Node status/storage health checks for both cluster nodes.

The node-detail records were dated 09:49 IST, while the Updates summary retained a 09:57 IST health-check time. A fresh Check again operation was initiated from the readiness page.

Node-side checks then showed:

| Component | Observed state before installation |
| --- | --- |
| AzLHOST1 and AzLHOST2 | Up |
| Infrastructure_1, UserStorage_1 and UserStorage_2 CSVs | Online |
| ClusterPerformanceHistory, Infrastructure_1, UserStorage_1 and UserStorage_2 virtual disks | Healthy, OK, DetachedReason None |

During the health check, Get-SolutionUpdateEnvironment reported HealthState InProgress and HealthCheckDate 1/1/0001. This placeholder was not treated as evidence of a clock fault. Environment State briefly read AppliedSuccessfully while CurrentVersion remained 12.2608.1003.8; it was not interpreted as installation of the new version.

The completed result was HealthState Success, State UpdateAvailable and CurrentVersion 12.2608.1003.8. The raw HealthCheckDate was 3 September 2026 07:31:41; the later portal displayed the completed check at 13:01 IST and readiness Healthy. Preserve the timestamp context of each surface rather than assuming every PowerShell field uses the node's local timezone.

The refreshed system-health check cleared the previous critical readiness result without a new storage repair. This establishes recovery for that check, not a guaranteed root cause for all earlier faults.

## 4. Update package and preparation

| Item | Verified value |
| --- | --- |
| Installed version before update | 12.2608.1003.8 |
| Selected solution update | 2026.08 Feature Update, 12.2608.1003.9 |
| Platform component | 12.2608.0.3020 |
| Services component | 10.2608.0.11 |
| Estimated package size | 5 GB |
| Scope | One system: localboxcluster |
| Reboot requirement displayed | Unknown; not a no-reboot guarantee |

The operator used the Azure Local resource's Updates > Prepare workflow. Readiness passed with two Informational entries, Cluster-Aware Updating Health Check Rules 10 and 11, and no blocking errors. Their detailed remediation text was not captured; no configuration change was made solely because they were listed.

The Review + prepare page explicitly stated that it would download and validate update content and run health checks. Observed progression:

1. Downloading.
2. HealthChecking, with UpdateStateProperties reporting 43% complete and HealthState InProgress.
3. Prepared in Azure Portal.

The preparation history recorded 13:13:09 to 13:30:10 IST, approximately 17 minutes. This is preparation duration only, not installation duration or a production estimate.

## 5. Installation checkpoint: IN PROGRESS

After preparation, the operator selected Install now, passed readiness again, reviewed the single-system Platform/Services update and confirmed final Install.

The reviewed installation run started at 14:45 IST. The latest supplied progress screenshot contained successful preparation/bootstrap, servicing-stack preparation, orchestrator credential/share updates, security-policy updates, Trusted Hosts updates and several software-dependency steps. Another InstallPowerShellCore step was In progress from 15:05 IST.

Update Moc pre-SSU was Skipped. Later steps, including orchestrator updates, secret rotation and cluster update, showed Unknown with no start time. No Failed step was visible in that screenshot. This does not prove that the full installation will succeed or explain why the skipped step was skipped.

**Current conclusion:** preparation complete; installation started and still in progress at the latest operator checkpoint. No final Installed/Success result, installed version 12.2608.1003.9 or post-update health evidence has been supplied.

While the run is active, leave the outer LocalBox host and nested environment powered on. Do not initiate competing updates, manual node restarts, storage moves or unrelated cluster changes. Read-only progress inspection and documentation can continue.

## 6. Reusable checks and completion gates

Run these read-only status commands directly on an Azure Local node using the deployment administrator account:

```powershell
Get-Date -Format o

Get-SolutionUpdate |
    Where-Object { $_.Version -eq '12.2608.1003.9' } |
    Format-List Version, State, UpdateStateProperties, HealthState

Get-SolutionUpdateEnvironment |
    Format-List CurrentVersion, HealthState, HealthCheckDate, State
```

Blank UpdateStateProperties does not establish a failed or stalled download. Inspect the current run's step details and timestamps in Updates > History if progress needs investigation. Do not start another run simply because a portal view is delayed.

**Pending, not executed as post-update validation:** after installation reports completion, verify all of the following:

1. The selected update reports Installed and its run has a successful terminal result.
2. CurrentVersion is 12.2608.1003.9 and system health is successful.
3. Both cluster nodes are Up; all three CSVs are Online; all virtual disks are Healthy/OK; storage jobs are reviewed.
4. QuorumResource is Cloud Witness; the witness and Cluster Group are Online.
5. Azure connectivity and recent monitoring data are present.
6. The workload VM is running and its guest/application access is verified.
7. Update the runbook, dependency/status tables and this record with actual final evidence.

Use [runbook section 21.1](06_Azure_LocalBox_Deployment_Runbook.md#211-resume-an-existing-lab-and-troubleshoot-cloud-witness) for read-only node/storage/quorum checks, not as authorization to rerun its conditional recovery commands. Record any failure and investigate the exact step before retrying.

## 7. Open items

- Installation completion and post-update validation.
- Network metric coverage and any additional monitoring demonstration required by the final accreditation evidence review.
- Permanent governance resolution; the existing expiry reminder remains unchanged.
- Final sanitized evidence packaging and operator laptop repository synchronization.

## 8. Microsoft references

- [Monitor Azure Local with platform metrics](https://learn.microsoft.com/en-us/azure/azure-local/manage/monitor-cluster-with-metrics)
- [Manage alert instances and user response](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-manage-alert-instances)
- [Azure Local update management and readiness checks](https://learn.microsoft.com/en-us/azure/azure-local/update/azure-update-manager-23h2)
- [Troubleshoot update health checks](https://learn.microsoft.com/en-us/azure/azure-local/update/update-troubleshooting-23h2)
- [Update phases and progress](https://learn.microsoft.com/en-us/azure/azure-local/update/update-phases-23h2)
- [PowerShell update status and verification](https://learn.microsoft.com/en-us/azure/azure-local/update/update-via-powershell-23h2)
- [Release notes and known issues](https://learn.microsoft.com/en-us/azure/azure-local/known-issues?view=azloc-2608)
