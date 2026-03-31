# Procedure 05 — Switch and VLAN Procedures

**Category:** Network Infrastructure  
**ITIL 4 Priority:** P1 (switch down affects multiple users) / P3 (single port issue) 

---

## VLAN Overview (from Lab Environment)

| VLAN ID | Name | Purpose | Subnet |
|---------|------|---------|--------|
| VLAN 10 | Work | Corporate endpoints, domain-joined devices | 10.10.10.0/24 |
| VLAN 20 | Guest / IoT | Guest Wi-Fi, IoT devices, untrusted | 10.10.20.0/24 |
| VLAN 99 | Management | Network device management interfaces | 10.10.99.0/24 |

**Inter-VLAN routing:** Handled by pfSense. Firewall rules block VLAN 20 from reaching VLAN 10 (lateral movement prevention).

---