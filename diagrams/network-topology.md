# Network Topology — Home Lab & Reference Architecture

## Lab Network Diagram (ASCII)

```
                        ┌─────────────────────────────┐
                        │         INTERNET              │
                        └──────────────┬───────────────┘
                                       │ WAN (DHCP from ISP)
                        ┌──────────────▼───────────────┐
                        │   pfSense Firewall (VM)        │
                        │   WAN: DHCP from ISP           │
                        │   LAN: 10.10.0.1/24            │
                        │   VLAN 10: 10.10.10.1/24       │
                        │   VLAN 20: 10.10.20.1/24       │
                        │   VLAN 99: 10.10.99.1/24       │
                        └──────────────┬───────────────┘
                                       │ Trunk (all VLANs)
                        ┌──────────────▼───────────────┐
                        │   Managed Switch (Virtual)     │
                        │   802.1Q VLAN Tagging          │
                        └──┬──────────┬────────────┬───┘
                           │          │            │
                    VLAN 10│   VLAN 20│    VLAN 99 │
              ┌────────────▼─┐  ┌─────▼──────┐  ┌─▼──────────────┐
              │  Windows DC  │  │ Guest/IoT  │  │  Management     │
              │  Server 2022 │  │  Devices   │  │  Interface      │
              │  AD, DNS,    │  │  No corp   │  │  Admin access   │
              │  DHCP        │  │  access    │  │  only           │
              └────┬─────────┘  └────────────┘  └────────────────┘
                   │
          ┌────────┴────────┐
          │                 │
   ┌──────▼──────┐  ┌───────▼──────┐
   │  Win 10 VM  │  │  Ubuntu VM   │
   │  Domain     │  │  Linux admin │
   │  joined     │  │  & Bash lab  │
   └─────────────┘  └──────────────┘
```

## WireGuard VPN Overlay

```
Remote Admin ──[WireGuard Encrypted Tunnel]──► pfSense WireGuard Server
                                               │
                                               ├── Peer: Remote-Admin-01
                                               │   Allowed IPs: 10.10.10.0/24
                                               │
                                               └── VLAN 10 access granted
```

## VLAN Firewall Rule Summary

| Source | Destination | Rule | Reason |
|--------|------------|------|--------|
| VLAN 10 | WAN | Allow | Corporate internet access |
| VLAN 10 | VLAN 20 | Block | Prevent lateral movement |
| VLAN 10 | VLAN 99 | Allow (admin IPs only) | Management access |
| VLAN 20 | WAN | Allow | Guest internet |
| VLAN 20 | VLAN 10 | Block | Isolation |
| VLAN 20 | VLAN 99 | Block | No management access |
```

---