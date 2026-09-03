# Azure Local VM, Windows Admin Center, Storage and Disk Validation

## Purpose

This document records the next verified implementation checkpoint for Accreditation Activity 3 after the logical network and Marketplace image were validated.

It captures only actions that were actually executed and verified in the LocalBox lab. It also records operational lessons learned while tracing the first Azure Local workload VM from Azure management down to the local Hyper-V and Storage Spaces Direct layers.

This document is safe for the public accreditation repository. Passwords, subscription IDs, tenant IDs, VM GUIDs, public IP addresses, and unsanitized screenshots are intentionally excluded.

---

## 1. First Azure Local Workload VM Creation

The first Azure Local workload VM was created successfully through Azure Portal.

Validated configuration:

```text
VM name                 azlocal-ws2025-01
VM kind                 Azure Local
Custom location         jumpstart
Image                    ws2025-azureedition-smalldisk
vCPU                     4
Memory                   8192 MB
Dynamic memory           Disabled
Guest management         Enabled at creation
Domain join              Disabled
Workload logical network localbox-vm-lnet-vlan200
IPv4 allocation          Static / automatic allocation
```

Backend validation confirmed:

```text
Provisioning state : Succeeded
Power state        : Running
Host node          : AzLHOST1
Operation          : Succeeded
Error              : none
```

Windows shortened the guest computer name because of the Windows hostname length limit. This did not indicate a provisioning failure.

Status: **PASS**.

---

## 2. LocalBox Management Layers and Administrative Identities

The lab uses different administrative identities for different management layers.

| Management layer | Administrative identity | Purpose |
| --- | --- | --- |
| Outer Azure IaaS VM `LocalBox-Client` | `.\arcdemo` | RDP and local administration of the LocalBox jump host |
| Nested Azure Local environment | `jumpstart\LocalBoxDeployUser` | Administrative access to `AzLHOST1`, `AzLHOST2`, and the nested Azure Local cluster |

The outer-VM username is defined by the pinned Microsoft LocalBox host deployment source. The nested deployment username is defined in the pinned `LocalBox-Config.psd1` source.

Pinned Microsoft references:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/bicep/host/host.bicep

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

Passwords must never be stored in the repository.

---

## 3. RDP Access to LocalBox-Client

`LocalBox-Client` has a public IP because the validated LocalBox deployment did not use Azure Bastion.

Initial TCP testing showed that RDP on TCP 3389 was not reachable from the administrator workstation.

Inspection confirmed:

- The public IP was attached to `LocalBox-Client-NIC`.
- No NSG was attached directly to the NIC.
- `LocalBox-NSG` was attached to `LocalBox-Subnet`.
- The NSG had no custom inbound rule permitting TCP 3389 from the Internet.
- The default `DenyAllInBound` rule therefore blocked the connection.

The pinned Microsoft `network.bicep` attaches `LocalBox-NSG` to `LocalBox-Subnet` and defines an empty custom security-rule list. The pinned post-provision script adds an RDP NSG rule only when the configured RDP port differs from 3389.

For controlled lab access, RDP should be allowed only from an approved administrator source IP rather than opening TCP 3389 to all Internet sources.

Status: **ROOT CAUSE VERIFIED**.

---

## 4. Windows Admin Center Installation and Certificate Position

Windows Admin Center was installed on `LocalBox-Client` and used successfully for cluster management.

For the temporary LocalBox lab, the installation used the self-signed certificate option. This is acceptable for a short-lived lab but is not the recommended production certificate model.

Production position:

- Use a certificate issued by a trusted enterprise or approved public CA.
- Match the certificate CN/SAN to the Windows Admin Center FQDN.
- Include the Server Authentication EKU.
- Protect the private key and implement certificate renewal before expiry.
- Prefer a controlled management gateway design rather than relying on a temporary self-signed certificate.

Windows Admin Center does not maintain a separate application-specific username/password database for this use case. Windows/AD credentials are used according to the target being managed.

---

## 5. Adding the Azure Local Cluster to Windows Admin Center

The short cluster name did not resolve from `LocalBox-Client`:

```text
localboxcluster -> DNS name does not exist
```

The FQDN resolved successfully:

```text
localboxcluster.jumpstart.local -> 192.168.1.100
```

Therefore Windows Admin Center was connected using:

```text
Cluster name : localboxcluster.jumpstart.local
Credential   : jumpstart\LocalBoxDeployUser
```

This distinction is operationally important. If the cluster short name is not resolvable from the management computer, use the resolvable FQDN rather than assuming a credential failure.

Status: **PASS**.

---

## 6. Windows Admin Center Cluster-Level Storage View

After connecting to `localboxcluster.jumpstart.local`, Windows Admin Center exposed the cluster-level views required to inspect Azure Local storage:

```text
Cluster resources
  Virtual machines
  Servers
  Volumes
  Drives
```

This is different from opening a single server connection, which primarily shows that individual server's storage view.

In the cluster connection, the `Volumes` and `Drives` tools provide the practical GUI view of the Storage Spaces Direct-backed cluster storage.

---

## 7. S2D / Cluster Drive Inventory

Windows Admin Center `Drives -> Inventory` showed the capacity drives participating in storage pool `SU1_Pool`.

Verified observations:

- 14 drives were visible.
- Drives were distributed across `AzLHOST1` and `AzLHOST2`.
- Status was healthy/OK.
- Drives were used for capacity.
- The LocalBox model was shown as `Virtual Disk`.
- The lab displayed HDD media type for most capacity devices.

Because this is a nested LocalBox environment, these are virtual disks presented to the nested Azure Local nodes. They are not production physical SSD/HDD devices.

Logical lab layering:

```text
Outer Azure VM data disks
        |
        v
Nested Azure Local node disks
        |
        v
Storage Spaces Direct pool: SU1_Pool
        |
        v
Cluster volumes / CSVs
        |
        v
Workload VM VHD/VHDX files
```

Status: **PASS**.

---

## 8. Cluster Volume Inventory

Windows Admin Center `Volumes -> Inventory` showed four healthy volumes:

| Volume | File system | Resiliency | Approximate size | Current owner shown in WAC |
| --- | --- | --- | ---: | --- |
| `ClusterPerformanceHistory` | ReFS | Two-way mirror | 20 GB | AzLHOST1 |
| `Infrastructure_1` | CSVFS_ReFS | Two-way mirror | 252 GB | AzLHOST2 |
| `UserStorage_1` | CSVFS_ReFS | Two-way mirror | 679 GB | AzLHOST2 |
| `UserStorage_2` | CSVFS_ReFS | Two-way mirror | 679 GB | AzLHOST1 |

All volumes were backed by storage pool `SU1_Pool` and reported healthy status.

Important operational distinction:

The node shown as the volume owner/coordinator does not have to be the node currently running a VM that consumes that CSV. Cluster Shared Volumes are accessible across cluster nodes.

Status: **PASS**.

---

## 9. Workload VM Placement in Windows Admin Center

Windows Admin Center `Virtual machines -> Inventory` showed `azlocal-ws2025-01` under `AzLHOST1`.

The VM details view confirmed:

```text
State             Running
Host              AzLHOST1
Virtual processors 4
Memory assigned   8 GB
Generation        2
Clustered         Yes
Status            Operating normally
```

This independently corroborated the Azure backend result that the VM was running on `AzLHOST1`.

Status: **PASS**.

---

## 10. Why Azure Portal Showed Zero Disks

Azure Portal showed the workload VM with `Disks = 0`, and the Azure Local CLI disk-list operation returned no standalone disk resources.

This did not mean the VM had no operating-system disk. The VM was running successfully and host-side Hyper-V inspection showed its boot disk attachment.

The practical lesson is that the Arc-enabled Azure Local VM resource model and UI do not necessarily surface an image-backed OS disk as a separate `Microsoft.AzureStackHCI/virtualHardDisks` resource in the same way an explicitly managed data disk is surfaced.

Do not interpret `Disks = 0` as proof that the VM has no boot disk.

---

## 11. Host-Side OS Disk Discovery

`LocalBox-Client` could reach WinRM on `AzLHOST1`, but the local `arcdemo` account was not authorized for administrative remoting to the Azure Local node.

Using the nested deployment account succeeded:

```text
jumpstart\LocalBoxDeployUser
```

The workload VM disk attachment was then inspected remotely with Hyper-V PowerShell.

Validated command pattern:

```powershell
$cred = Get-Credential

Invoke-Command `
  -ComputerName AzLHOST1 `
  -Credential $cred `
  -ScriptBlock {
      Get-VM -Name "azlocal-ws2025-01" |
          Get-VMHardDiskDrive |
          Select-Object ControllerType, ControllerNumber, ControllerLocation, Path
  }
```

Verified attachment:

```text
ControllerType     SCSI
ControllerNumber   0
ControllerLocation 0
Path               C:\ClusterStorage\UserStorage_1\<resource-folder>\azlocal-ws2025-01-<redacted>-OSDisk-<redacted>.vhd
```

The unique identifiers embedded in the actual generated path are intentionally redacted from public documentation.

This directly proves that the OS disk file is logically placed on `UserStorage_1`.

Status: **PASS**.

---

## 12. OS Disk Format, Type and Capacity

The attached boot disk was inspected with `Get-VHD`.

Validated command pattern:

```powershell
Invoke-Command `
  -ComputerName AzLHOST1 `
  -Credential $cred `
  -ScriptBlock {
      $disk = Get-VM -Name "azlocal-ws2025-01" |
          Get-VMHardDiskDrive |
          Select-Object -First 1

      Get-VHD -Path $disk.Path |
          Select-Object Path, VhdType, VhdFormat, FileSize, Size, ParentPath
  }
```

Verified result:

```text
VhdType    Fixed
VhdFormat  VHD
Size       approximately 30 GiB
ParentPath empty
```

Important findings:

- The image-backed OS disk in this LocalBox VM is a fixed VHD, not VHDX.
- `ParentPath` was empty, so the attached boot disk was not a differencing disk.
- File size and virtual size were effectively the same, which is consistent with a fixed VHD.

Status: **PASS**.

---

## 13. Mapping the OS Disk to the Cluster Storage Layers

The OS disk path contains:

```text
C:\ClusterStorage\UserStorage_1\...
```

That path is a Cluster Shared Volume namespace, not a statement that the workload data is stored on the local system drive of `AzLHOST1`.

The validated mapping is:

```text
azlocal-ws2025-01
        |
        v
OS VHD attachment
        |
        v
C:\ClusterStorage\UserStorage_1\...
        |
        v
CSV / volume: UserStorage_1
        |
        v
Virtual disk: UserStorage_1
        |
        v
Resiliency: Mirror / Two-way mirror
        |
        v
S2D storage pool: SU1_Pool
        |
        v
Underlying capacity drives across the cluster
```

This explains why the VM can run on `AzLHOST1` while Windows Admin Center shows the current owner/coordinator of `UserStorage_1` as `AzLHOST2`.

VM compute placement and CSV ownership are separate cluster concepts.

---

## 14. PowerShell Storage Validation

PowerShell independently verified the same storage picture.

Cluster Shared Volumes included:

```text
Infrastructure_1
UserStorage_1
UserStorage_2
```

Virtual disks showed healthy mirror resiliency. `UserStorage_1` and `UserStorage_2` were approximately 679 GiB each, and `Infrastructure_1` was approximately 252 GiB.

`Get-PhysicalDisk` showed the multiple LocalBox capacity devices backing the nested storage environment.

This provided a command-line cross-check of the Windows Admin Center GUI findings.

Status: **PASS**.

---

## 15. Windows Admin Center VHD Display Limitation Observed

Windows Admin Center successfully displayed the VM and its host, CPU, memory, generation, cluster status, and operating-system information.

However, the VM `VHDs` section returned an extension/UI error and did not display the virtual disks.

The observed error indicated that Windows Admin Center could not load the virtual-disk properties for this VM.

This was treated as a management-view limitation, not a VM/storage failure, because:

- The VM was running normally.
- Azure backend provisioning had succeeded.
- Hyper-V PowerShell returned the disk attachment.
- `Get-VHD` returned valid disk metadata.
- The backing CSV and S2D volumes were healthy.

Operational lesson:

> When a management GUI does not expose the required disk detail, verify the VM at the authoritative local virtualization/storage layer before concluding that the disk is missing or unhealthy.

---

## 16. OS Disk Expansion Management Path

Microsoft's current supported-operations guidance for Azure Local VMs differentiates OS-disk and data-disk expansion.

For Azure Local VMs enabled by Azure Arc:

- **Expand an OS disk** is listed as a supported **local-tools** operation. Local tools include PowerShell, Windows Admin Center, Hyper-V Manager, Failover Cluster Manager, and Virtual Machine Manager.
- **Expand a data disk** is supported through Azure CLI in the applicable Azure Local versions.

Microsoft reference:

https://learn.microsoft.com/en-us/azure/azure-local/manage/virtual-machine-operations

Therefore GUI access is not mandatory for an OS-disk expansion. PowerShell can be used when the operation is performed according to Microsoft's supported local-tools boundary.

Important control:

Do not blindly use local tools for other Arc-managed VM operations. Microsoft explicitly separates operations that must be performed through Azure Portal/Azure CLI from the smaller set of operations supported through local tools. Unsupported local changes can create synchronization problems between the local VM state and Azure.

The lab did **not** resize the OS disk at this checkpoint. Only the discovery and supported management path were validated.

---

## 17. Production Storage Interpretation

The LocalBox storage layout is useful for learning, but the underlying drives in this environment are nested virtual devices backed by the outer Azure IaaS VM.

Production Azure Local uses supported physical hardware and a validated storage design. Production decisions must therefore consider real media type, hardware fault domains, capacity reserve, resiliency, IOPS, throughput, latency, rebuild behavior, and lifecycle requirements.

Do not use LocalBox storage performance as a production benchmark.

---

## 18. Current Activity 3 Checkpoint

Status synchronized on **3 September 2026** with the [VM lifecycle evidence](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) and [monitoring and update evidence](11_Azure_Monitoring_and_Platform_Update_Validation.md). PASS refers to recorded operator evidence, not a new live test by this documentation update. Platform installation to 12.2608.1003.9 and the agreed post-update validation are PASS. Fresh node, CSV, virtual-disk, witness/quorum, guest-login and basic monitoring evidence is recorded in docs/11. This is a point-in-time lab result, not production certification.

| Implementation checkpoint | Status |
| --- | --- |
| LocalBox host deployment | PASS |
| Azure Local cluster deployment | PASS |
| Arc / Azure connectivity | PASS |
| Logical workload network | PASS |
| Marketplace VM image | PASS |
| Azure Local workload VM creation | PASS |
| VM backend provisioning | PASS |
| VM running state | PASS |
| VM host-node placement verification | PASS |
| WAC installation and access | PASS |
| WAC cluster connection | PASS |
| S2D drive inventory | PASS |
| S2D / CSV volume inventory | PASS |
| OS disk host-side discovery | PASS |
| OS disk format/type/capacity validation | PASS |
| OS disk to CSV/S2D mapping | PASS |
| OS disk expansion support path understood | PASS, no resize executed |
| NIC / workload IP validation | PASS |
| Guest-management verification | PASS |
| Workload VM lifecycle start/stop/restart | PASS |
| Azure monitoring and management validation | PASS (basic scope; see docs/11) |
| Azure Local platform update/lifecycle validation | PASS — 12.2608.1003.9 and post-update checks verified |

---

## Next Required Step

The subsequent [VM network, guest-management and lifecycle evidence](09_Azure_Local_VM_Network_Guest_Management_and_Lifecycle_Validation.md) confirms NIC/IP, guest management, and VM stop/start/restart are complete. The [lab recovery checkpoint](10_Lab_Recovery_Checkpoint_and_Follow_Up.md) records the later startup and witness recovery.

Installation and post-update storage/workload checks are now PASS. See the [monitoring and update evidence](11_Azure_Monitoring_and_Platform_Update_Validation.md) for the fresh results, guest-login recovery and remaining limitations. Continue with final accreditation content review.

The VM should not be resized or otherwise changed merely to demonstrate capability unless that change is required by the accreditation task.

