# Azure Monitoring and Platform Update Validation

**Final operator-evidence checkpoint: 3 September 2026. Installation and agreed post-update validation PASS.**

## 1. Purpose and evidence boundary

This record continues [VM lifecycle validation](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) and the [recovery checkpoint](10_Lab_Recovery_Checkpoint_and_Follow_Up.md) for Activity 3 of the [governing accreditation scope](00_Accreditation_Scope_and_Customer_Scenario_Source_of_Truth.md).

Evidence consists of operator-supplied Azure Portal screenshots and node-side PowerShell output reviewed during the session. This is a sanitized text record, not a claim of direct access to the running cluster. Raw screenshots, tenant/subscription identifiers, account details and operation identifiers have not been published.

All results are point-in-time observations. Sections 2–4 preserve pre-update history; sections 5–6 record installation completion and fresh post-update evidence separately. The nested Australia East LocalBox environment is not a production performance benchmark.

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

## 5. Installation completion: PASS

After preparation, the operator selected Install now, passed readiness again, reviewed the single-system Platform/Services update and confirmed installation.

Early screenshots showed successful preparation/bootstrap and servicing-stack work, an in-progress dependency task, and Unknown on unstarted downstream tasks. A subsequent screenshot confirmed servicing-stack, platform-component and OS-update success; security-settings work was then in progress. These were intermediate checkpoints, not terminal failures.

The final portal screenshot explicitly reported **All updates are installed successfully**. Download, readiness validation and installation were Completed. The Start update task returned Success with portal timestamps **3 September 2026, 14:45 to 17:49 IST**, approximately **3 hours 4 minutes**. The separate 17-minute preparation duration must not be described as installation time. A prior operator estimate of nearly five hours is not the measured install duration.

Node-side verification on AzLHOST1 then returned:

```text
CurrentVersion  : 12.2608.1003.9
HealthState     : Success
HealthCheckDate : 9/3/2026 12:29:05 PM
State           : AppliedSuccessfully
```

HealthCheckDate is retained exactly as returned; its field did not include a timezone. No unsupported timezone conversion is used.

## 6. Post-update validation: PASS within the recorded scope

| Check | Supplied post-update evidence | Result |
| --- | --- | --- |
| Nodes | AzLHOST1 and AzLHOST2 Up | PASS |
| Cluster shared volumes | Infrastructure_1 and UserStorage_1 Online on AzLHOST2; UserStorage_2 Online on AzLHOST1 | PASS |
| Virtual disks | ClusterPerformanceHistory, Infrastructure_1, UserStorage_1 and UserStorage_2 Healthy / OK | PASS |
| Storage jobs | Get-StorageJob returned no rows | No jobs returned |
| Cluster resources | Cloud Witness and Cluster Name Online on AzLHOST2 | PASS for these resources |
| Quorum | QuorumType Majority; QuorumResource Cloud Witness | PASS |
| Azure Local registration | Clustered, Registered, Connected | Connection snapshot verified |
| Workload VM | Running / Operating normally in WAC; Running in Azure portal | PASS |
| Workload Arc agent | Enabled (connected) in Azure Overview | PASS |
| Interactive guest access | Logged-in Windows desktop visible through WAC VMConnect | PASS after password recovery |
| Recent basic monitoring | CPU, memory, disk-operation and disk-byte charts populated in the later monitoring checkpoint | PASS |
| Last-hour alerts view | No alert instances listed; recommended-alert-rule setup prompt visible | Observation only; not proof of historical-alert resolution |

The latest monitoring charts displayed approximately 16.30% average CPU and 20.9 GB memory for the selected window. Treat chart legends as displayed aggregates, not exact instantaneous readings or production benchmarks. Network metric legends remained `--`; network-metric coverage is unverified, not confirmed zero traffic.

Get-AzureStackHCI retained LastConnected `9/2/2026 10:00:01 PM`; the later portal showed Connected—8 hours ago and Health status `---`. These are retained caveats: do not call the connection timestamp a new heartbeat or the empty portal field a health PASS. Separate fresh backend health and populated metric evidence support the recorded validation. The cause and closure state of the earlier 32 heartbeat alerts remain unproven. No alert rules were enabled, disabled or manually closed for this check.

### Guest-access recovery during validation

The guest console was reachable, but the operator had forgotten the VM-local password. This was separate from the successful platform update. Read-only Azure Arc Run Command operations enumerated local users and administrator-group membership. A targeted local-account reset used hidden input and ProtectedParameter; execution returned Succeeded, exit code 0 and no error. A subsequent WAC VMConnect screenshot showed the logged-in desktop.

No password, full account identifier, protected value, raw command payload, subscription identifier or recovery operation ID is included here. No VM rebuild, host password reset, or NLA disablement was required. An Activate Windows watermark remained visible; guest activation was not verified or repaired in this task.

### Reusable verification context

Run cluster and update cmdlets on an Azure Local node, not on LocalBox-Client. Confirm hostname first. Azure Arc Run Command operations in this session ran from Azure Cloud Shell PowerShell. WAC VMConnect is the guest console, while Azure portal Overview is the resource-management view.

```powershell
hostname
Get-SolutionUpdateEnvironment |
    Format-List CurrentVersion, HealthState, HealthCheckDate, State
Get-ClusterNode | Format-Table Name, State -AutoSize
Get-ClusterSharedVolume | Format-Table Name, State, OwnerNode -AutoSize
Get-VirtualDisk | Format-Table FriendlyName, HealthStatus, OperationalStatus -AutoSize
Get-StorageJob | Format-Table Name, JobState, PercentComplete -AutoSize
Get-ClusterResource |
    Where-Object { $_.Name -in @('Cloud Witness', 'Cluster Name') } |
    Format-Table Name, State, OwnerNode -AutoSize
Get-ClusterQuorum | Format-List QuorumType, QuorumResource
Get-AzureStackHCI |
    Format-List ClusterStatus, RegistrationStatus, ConnectionStatus, LastConnected
```

These are reusable read-only checks, not a request to rerun the completed update or password reset. Earlier Cluster Group Online evidence belongs to the recovery checkpoint; the post-update resource output specifically verified Cloud Witness and Cluster Name.

**Completion conclusion:** Activity 3 platform update and the agreed post-update checks are complete at this evidence checkpoint. Final deck approval, customer production readiness, application-level business testing and overall accreditation acceptance are not implied.

## 7. Remaining follow-ups

- Permanent governance alignment before the existing waiver expiry; reminder unchanged.
- Guest Windows activation verification.
- Network-metric coverage. Advanced monitoring, alert configuration/delivery tests and permanent resolution of historical heartbeat alerts are not claimed.
- Review/finalize presentation and migration-plan drafts, customer-meeting preparation and the eight written validation answers once supplied.
- Synchronize the operator's laptop repository safely; this remote publication does not update it.

## 8. Microsoft references

- [Monitor Azure Local with platform metrics](https://learn.microsoft.com/en-us/azure/azure-local/manage/monitor-cluster-with-metrics)
- [Manage alert instances and user response](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-manage-alert-instances)
- [Azure Local update management and readiness checks](https://learn.microsoft.com/en-us/azure/azure-local/update/azure-update-manager-23h2)
- [Troubleshoot update health checks](https://learn.microsoft.com/en-us/azure/azure-local/update/update-troubleshooting-23h2)
- [Update phases and progress](https://learn.microsoft.com/en-us/azure/azure-local/update/update-phases-23h2)
- [PowerShell update status and verification](https://learn.microsoft.com/en-us/azure/azure-local/update/update-via-powershell-23h2)
- [Release notes and known issues](https://learn.microsoft.com/en-us/azure/azure-local/known-issues?view=azloc-2608)

- [Azure Arc Run Command and protected parameters](https://learn.microsoft.com/en-us/azure/azure-arc/servers/run-command)
- [WAC VMConnect](https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines#manage-a-virtual-machine-through-the-hyper-v-host-vmconnect)

