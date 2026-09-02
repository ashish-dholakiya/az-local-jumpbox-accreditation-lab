# Azure Local VM Network, Guest Management and Lifecycle Validation

## Purpose

This document records the next verified accreditation checkpoint after `docs/08_Azure_Local_VM_WAC_Storage_and_Disk_Validation.md`.

It captures only actions that were actually executed and verified in the LocalBox lab. Sensitive identifiers such as subscription IDs, tenant IDs, operation IDs, user email addresses, VM GUIDs, public IP addresses, passwords, and unsanitized screenshots are intentionally excluded from this public repository.

---

## 1. Azure Local VM NIC and Workload IP Validation

The first Azure Local workload VM uses the NIC resource:

```text
azlocal-ws2025-01-nic
```

Backend inspection with the Azure Local CLI confirmed:

```text
Provisioning state : Succeeded
Private IP         : 192.168.200.2
Prefix length      : /24
Gateway            : 192.168.200.1
Logical network    : localbox-vm-lnet-vlan200
NSG                 : none on the Azure Local NIC resource
Operation status    : Succeeded
```

This confirms that the VM NIC is successfully attached to the intended Azure Local workload logical network and received an address from the `192.168.200.0/24` workload subnet.

Status: **PASS**.

---

## 2. Microsoft LocalBox Workload Network Source Validation

The pinned Microsoft LocalBox source defines the workload network as:

```text
vmGateway  = 192.168.200.1
vmIpPrefix = 192.168.200.0/24
vmDNS      = 192.168.1.254
vmVLAN     = 200
```

The pinned `Configure-VMLogicalNetwork.ps1` uses those values to create:

```text
localbox-vm-lnet-vlan200
```

with static allocation on:

```text
ConvergedSwitch(compute_management)
```

The LocalBox configuration also defines the nested router interface for workload VLAN 200 as:

```text
BGPRouterIP_VLAN200 = 192.168.200.1/24
```

Pinned Microsoft references:

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/LocalBox-Config.psd1

https://github.com/microsoft/azure_arc/blob/3f433866757688d926ae6707e9c0041d8e640b82/azure_jumpstart_localbox/artifacts/PowerShell/Configure-VMLogicalNetwork.ps1

Status: **SOURCE-ALIGNED**.

---

## 3. Direct RDP Test from LocalBox-Client to the Workload VM

A connectivity test was attempted from the outer Azure IaaS VM `LocalBox-Client` to the workload VM address on TCP 3389.

Observed result:

```text
Source interface : outer LocalBox-Client Ethernet
Source network   : 172.16.1.0/24
Destination      : 192.168.200.2:3389
TCP result       : failed
Ping result      : failed
```

The route table on `LocalBox-Client` was then inspected. It contained default routes through the outer Azure network and the LocalBox internal switch, but no explicit route for:

```text
192.168.200.0/24
```

Therefore the failed direct RDP test from the outer jump-host interface is not evidence that the Azure Local VM NIC or logical network failed.

The pinned LocalBox source defines the nested workload network and its router gateway, but it does not guarantee direct RDP reachability from the outer `LocalBox-Client` Azure interface to every workload VM.

Operational lesson:

> Validate the Azure Local workload network through the Azure Local NIC resource, logical-network configuration, guest-management state, and appropriate workload-side management paths. Do not treat failure of an unsupported or unconfigured cross-layer RDP path as a workload-network failure.

Status: **EXPECTED LAB NETWORK BEHAVIOR / NO REMEDIATION APPLIED**.

---

## 4. Guest Management and Azure Arc Validation

Guest management was enabled when `azlocal-ws2025-01` was created.

The raw `az stack-hci-vm show` status object exposed VM power and provisioning status but did not expose a guest-agent field in that response shape.

Raw status validation showed:

```text
Power state          : Running
Provisioning status  : Succeeded
Error code           : empty
Error message        : empty
```

Azure Portal was then used as the authoritative management view for the guest-management state.

Verified portal state:

```text
Guest management / Arc agent : Enabled (connected)
VM state                     : Running
```

The portal also displayed the connected agent version and the workload NIC/IP details.

Important lesson:

> A null value from a guessed or unavailable CLI property path must not be classified as guest-agent failure when the raw resource status does not expose that field. Verify the actual guest-management state through the supported Azure Local / Azure Arc management surface.

Status: **PASS**.

---

## 5. VM Stop Lifecycle Validation

The VM was stopped through Azure Local CLI:

```powershell
az stack-hci-vm stop `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "azlocal-ws2025-01"
```

The stop operation completed successfully.

Post-operation verification showed:

```text
PowerState        : Stopped
ProvisioningState : Succeeded
```

Status: **PASS**.

---

## 6. VM Start Lifecycle Validation

The same VM was started through Azure Local CLI:

```powershell
az stack-hci-vm start `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "azlocal-ws2025-01"
```

Post-operation verification showed:

```text
PowerState        : Running
ProvisioningState : Succeeded
```

Status: **PASS**.

---

## 7. VM Restart Lifecycle Validation

The running VM was restarted through Azure Local CLI:

```powershell
az stack-hci-vm restart `
  --resource-group "rg-azlocal-localbox-accreditation-aue" `
  --name "azlocal-ws2025-01"
```

Post-operation verification showed:

```text
PowerState        : Running
ProvisioningState : Succeeded
```

Status: **PASS**.

---

## 8. VM Lifecycle Conclusion

The required core lifecycle actions for the first Azure Local workload VM were executed successfully:

```text
Stop     PASS
Start    PASS
Restart  PASS
```

The final verified VM state was:

```text
PowerState        Running
ProvisioningState Succeeded
```

Therefore the Azure Local workload VM lifecycle checkpoint is complete.

Status: **PASS**.

---

## 9. Current Activity 3 Checkpoint

| Implementation checkpoint | Status |
| --- | --- |
| LocalBox host deployment | PASS |
| Azure Local cluster deployment | PASS |
| Arc / Azure connectivity | PASS |
| Logical workload network | PASS |
| Marketplace VM image | PASS |
| Azure Local workload VM creation | PASS |
| VM backend provisioning | PASS |
| VM host-node placement verification | PASS |
| Windows Admin Center access | PASS |
| S2D / CSV storage validation | PASS |
| OS disk host-side discovery | PASS |
| NIC / workload IP validation | PASS |
| Microsoft LocalBox workload-network source validation | PASS |
| Guest management / Azure Arc connection | PASS |
| VM stop | PASS |
| VM start | PASS |
| VM restart | PASS |
| Azure Local VM lifecycle | PASS |
| Azure monitoring and management validation | PENDING |
| Azure Local update / lifecycle validation | PENDING |

---

## 10. Next Required Step

The next controlled Activity 3 task is:

```text
Azure monitoring and management validation
```

Recommended next sequence:

```text
1. Review Azure Local cluster health and alerts.
2. Review workload VM management/health visibility.
3. Confirm Azure Arc connectivity and available management operations.
4. Capture only sanitized evidence required for accreditation.
5. Then validate Azure Local update and lifecycle management.
```

Do not expand scope beyond the accreditation requirements unless the additional work is explicitly recorded as further understanding rather than accreditation evidence.
