# Network Topology - Home Lab Architecture

This document describes the complete network architecture of the home lab used
to develop and validate all skills in this portfolio. The design mirrors a small
enterprise or Microsoft Partner client environment with VLAN segmentation,
pfSense firewall enforcement, Azure hybrid connectivity, and SIEM visibility.

---

## Physical/Logical Architecture

```
                        ┌─────────────────────────────────────────┐
                        │           INTERNET (ISP WAN)            │
                        └──────────────────┬──────────────────────┘
                                           │ WAN (DHCP from ISP)
                       ┌───────────────────▼────────────────────────┐
                       │           pfSense Firewall (VM)            │
                       │  WAN : DHCP from ISP                       │
                       │  VLAN 10 (Work)  : 10.10.10.1/24           │
                       │  VLAN 20 (Guest) : 10.10.20.1/24           │
                       │  VLAN 99 (Mgmt)  : 10.10.99.1/24           │
                       │  WireGuard VPN   : 10.10.200.0/24          │
                       └───────────────────┬────────────────────────┘
                                           │ 802.1Q Trunk (all VLANs)
                       ┌───────────────────▼────────────────────────┐
                       │      Virtual Managed Switch (Proxmox)      │
                       │      802.1Q VLAN tagging                   │
                       └──┬──────────────┬───────────────┬──────────┘
                          │              │               │
               VLAN 10 ───┘  VLAN 20 ───┘   VLAN 99 ───┘
                 │               │               │
    ┌────────────▼──────┐  ┌─────▼────────┐  ┌──▼────────────────┐
    │  Windows DC (dc01)│  │ Kali Linux   │  │  Splunk Server    │
    │  10.10.10.10      │  │ 10.10.20.10  │  │  10.10.99.10      │
    │  AD DS, DNS,      │  │ Pen testing  │  │  SIEM, log agg.   │
    │  DHCP, GPO        │  │ Isolated     │  │  Splunk Free      │
    └────────┬──────────┘  └──────────────┘  └───────────────────┘
             │
    ┌────────┴────────────────────────────────┐
    │                                         │
┌───▼──────────────┐              ┌──────────▼──────────────────┐
│  Windows 10 VM   │              │  Ubuntu 22.04 VM (ubuntu-   │
│  10.10.10.101    │              │  admin) 10.10.10.20         │
│  Domain-joined   │              │  Bash scripting, Python,    │
│  AD Connect host │              │  Splunk Fwd, Nessus scan src│
└──────────────────┘              └─────────────────────────────┘
                    │
                    │  Azure AD Connect sync
                    ▼
    ┌─────────────────────────────────────────────────────────────┐
    │                   MICROSOFT AZURE                           │
    │  Azure Entra ID (synced from contoso.local via AD Connect)  │
    │  Virtual Network: 10.20.0.0/16                              │
    │    ├─ subnet-servers: 10.20.1.0/24 (Windows Server VM)      │
    │    └─ subnet-linux:   10.20.2.0/24 (Ubuntu VM)              │
    │  Azure Monitor + Log Analytics Workspace                    │
    │  Microsoft Intune (M365 Dev Tenant)                         │
    └─────────────────────────────────────────────────────────────┘
```

---

## WireGuard VPN Configuration

WireGuard is configured on pfSense to allow secure remote admin access
to the lab without exposing management interfaces to the internet.

```
WireGuard Server on pfSense:
  Listen port   : 51820 (UDP)
  Server address: 10.10.200.1/24
  Interface     : wg0

Peer: Remote-Admin-Laptop
  Public key    : [generated on client]
  Allowed IPs   : 10.10.200.2/32
  Tunnel IP     : 10.10.200.2
  Access granted: VLAN 10 and VLAN 99 (management + corporate segment)

pfSense firewall rule for WireGuard interface:
  Allow wg0 source → 10.10.10.0/24 (VLAN 10 resources)
  Allow wg0 source → 10.10.99.0/24 (management segment)
  Block wg0 → 10.10.20.0/24 (guest/IoT not accessible via VPN)
```

---

## VLAN Firewall Rule Summary

| Source VLAN | Destination | Rule | Justification |
|-------------|------------|------|---------------|
| VLAN 10 (Work) | WAN (internet) | ✅ Allow | Corporate internet access |
| VLAN 10 (Work) | VLAN 20 (Guest) | ❌ Block | Prevent corporate → untrusted lateral movement |
| VLAN 10 (Work) | VLAN 99 (Mgmt) | ✅ Allow (admin IPs only) | Management access restricted to designated admin workstations |
| VLAN 20 (Guest) | WAN (internet) | ✅ Allow (HTTP/HTTPS only) | Guest internet access - limited to web browsing |
| VLAN 20 (Guest) | VLAN 10 (Work) | ❌ Block | Strict isolation - guest/IoT cannot reach corporate |
| VLAN 20 (Guest) | VLAN 99 (Mgmt) | ❌ Block | No management access for guest devices |
| VLAN 20 (Guest) | VLAN 20 (Guest) | ❌ Block | Prevent IoT device-to-device communication (lateral movement prevention) |
| VLAN 99 (Mgmt) | All VLANs | ✅ Allow | Management segment must reach everything to administer it |
| Any | VLAN 99 (Mgmt) | ❌ Block (except admin IPs) | Protect management plane from all unauthorised sources |

---

## Design Decisions - Why This Architecture

**Why VLAN 20 blocks IoT-to-IoT traffic:**
IoT devices are notoriously insecure. A compromised IoT device (smart bulb, IP camera)
should not be able to communicate with other IoT devices on the same segment — this
prevents a compromised device from being used as a pivot point. This is the client
isolation pattern used in enterprise guest Wi-Fi deployments.

**Why Splunk is on VLAN 99 (Management) rather than VLAN 10 (Work):**
The SIEM must receive logs from all segments without being reachable from the untrusted
VLAN 20 segment. Placing it in VLAN 99 means only authorised admin workstations can
access the Splunk web interface, while pfSense firewall rules allow log forwarding traffic
from all VLANs to the Splunk server on VLAN 99.

**Why WireGuard instead of OpenVPN:**
WireGuard has a significantly smaller codebase (~4,000 lines vs ~600,000 for OpenVPN),
making it easier to audit for security vulnerabilities. It also performs better on
low-spec hardware (relevant for a VM-based lab) and has faster handshake times, making
reconnection after brief network interruptions nearly seamless.


---