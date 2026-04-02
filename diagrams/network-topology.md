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