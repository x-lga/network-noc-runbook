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

## Procedure — Device Not Getting IP on Expected VLAN

```
1. Identify which port the device is connected to (check cable path or switch port label)
2. Confirm switch port is assigned to correct VLAN:
   - Managed switch CLI: show interfaces GigabitEthernet0/1 switchport
   - Check: Access VLAN = expected VLAN ID
3. Check DHCP scope for that VLAN has available leases
4. Check pfSense DHCP server: Services > DHCP Server > select VLAN interface
5. If lease pool exhausted: escalate for pool expansion
6. If port not in correct VLAN: reassign access port (requires change authorisation)
```

---

## Procedure — pfSense Firewall VLAN Rule Check

pfSense firewall rules for inter-VLAN are applied on the source interface.

```
VLAN 10 (Work) rules:
  - Allow VLAN 10 to WAN (internet)
  - Allow VLAN 10 to VLAN 99 (management) - restricted to admin IPs
  - Block VLAN 10 to VLAN 20

  VLAN 20 (Guest) rules:
  - Allow VLAN 20 to WAN (internet) only
  - Block VLAN 20 to everything else (including VLAN 10)

  Verification:
  pfSense > Diagnostics > Packet Capture (select interface, capture to confirm traffic flow)
  pfSense > Status > System Logs > Firewall (check blocked traffic)
```