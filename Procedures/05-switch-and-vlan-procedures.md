# Procedure 05 - Switch and VLAN Procedures

**Category:** Network Infrastructure  
**ITIL 4 Priority:** P1 (switch down, multiple users affected) | P3 (single port issue)  
**Cert alignment:** CompTIA Network+  
**Lab environment:** pfSense 2.7 firewall + virtual managed switch (802.1Q) on Proxmox  
**Last reviewed:** 2026-07

---

## Lab VLAN Architecture

This section documents the VLAN design implemented in the home lab environment.
It mirrors the architecture used in small-to-medium enterprise environments and
Microsoft Partner client deployments.

| VLAN ID | Name | Purpose | Subnet | Gateway |
|---------|------|---------|--------|---------|
| VLAN 10 | Work | Corporate endpoints - domain-joined devices, servers | 10.10.10.0/24 | 10.10.10.1 (pfSense) |
| VLAN 20 | Guest / IoT | Guest Wi-Fi, IoT devices, untrusted endpoints, Kali Linux lab | 10.10.20.0/24 | 10.10.20.1 (pfSense) |
| VLAN 99 | Management | Network device management interfaces, Splunk server | 10.10.99.0/24 | 10.10.99.1 (pfSense) |

**Design rationale:**
- VLAN 10 is isolated from VLAN 20 to prevent lateral movement from untrusted devices
- VLAN 20 has internet access only - it cannot reach VLAN 10 corporate resources
- VLAN 99 is accessible only from specific admin IPs - prevents management interface exposure
- pfSense handles all inter-VLAN routing and enforces the firewall rules between segments

---

## pfSense Firewall Rules - Inter-VLAN Policy

Rules in pfSense are applied on the **source** interface (the VLAN the traffic is coming from).

**VLAN 10 (Work) rules:**
```
Rule 1: Allow VLAN 10 → WAN (internet) - all traffic
Rule 2: Allow VLAN 10 → VLAN 99 - only from admin workstation IPs (10.10.10.10, 10.10.10.11)
Rule 3: Block VLAN 10 → VLAN 20 (prevent corporate devices accessing guest/IoT segment)
Rule 4: Allow VLAN 10 → VLAN 10 (allow intra-VLAN traffic for file shares, domain comms)
```

**VLAN 20 (Guest/IoT) rules:**
```
Rule 1: Allow VLAN 20 → WAN (internet) — HTTP/HTTPS only (port 80, 443)
Rule 2: Block VLAN 20 → VLAN 10 (strict isolation — no corporate access)
Rule 3: Block VLAN 20 → VLAN 99 (no management access)
Rule 4: Block VLAN 20 → VLAN 20 (prevent IoT devices communicating with each other)
```

**VLAN 99 (Management) rules:**
```
Rule 1: Allow VLAN 99 → WAN (for device firmware updates)
Rule 2: Allow VLAN 99 → all VLANs (management must reach everything)
Rule 3: Block any → VLAN 99 except admin IPs in VLAN 10 (protect management plane)
```

---

## Procedure - Device Not Getting an IP on the Expected VLAN

**Symptom:** User's computer shows APIPA address (169.254.x.x) or gets an IP from the
wrong subnet (e.g., gets 10.10.20.x when it should be 10.10.10.x).

**Diagnosis steps:**

1. **Identify which physical port the device is connected to**
   - Check the cable path to the patch panel
   - Note the switch and port number
   - Check the switch port label or documentation

2. **Check the switch port VLAN assignment**
   On a Cisco IOS managed switch:
   ```bash
   # Show VLAN assignment for a specific port
   show interfaces GigabitEthernet0/1 switchport
   # Look for: "Access Mode VLAN: X" — confirms which VLAN this port is in

   # Show all VLAN assignments at a glance
   show vlan brief
   ```
   On a pfSense virtual switch (lab): check the network bridge assignment in Proxmox.

3. **Check the DHCP scope for the expected VLAN**
   In pfSense: Services > DHCP Server > select the VLAN interface
   - Confirm the scope is enabled
   - Check that available leases exist (pool not exhausted)
   - Check the lease table for the device's MAC address

4. **If the port is on the wrong VLAN (requires change authorisation):**
   ```bash
   # On Cisco IOS - change access port to correct VLAN
   # REQUIRES: Change Management approval before executing in production
   configure terminal
   interface GigabitEthernet0/1
   switchport mode access
   switchport access vlan 10
   exit
   write memory
   ```

5. **After correcting the VLAN assignment:**
   On the client device: `ipconfig /release` → `ipconfig /renew`
   Confirm the device receives an IP from the correct subnet.

---

## Procedure - Switch Port Shows err-disabled

An err-disabled port has been automatically shut down by the switch due to a
detected error condition. The port appears completely inactive.

**Common causes of err-disabled:**
- Port security violation (unexpected MAC address seen on a MAC-restricted port)
- BPDU guard triggered (a device sent a BPDU - only switches should do this)
- Loop detection (a loop was detected on the port)
- PoE overcurrent (a PoE-powered device drew too much power)

**Procedure:**
```bash
# Identify err-disabled ports
show interfaces status | include err-disabled

# Check the reason
show interfaces GigabitEthernet0/X
# Look for: "err-disabled" in the line protocol status
# And: "Last clearing" timestamp to know when it was disabled

# Recover the port (after identifying and removing the cause)
configure terminal
interface GigabitEthernet0/X
shutdown
no shutdown
exit
write memory

# Verify port comes back up
show interfaces GigabitEthernet0/X status
```

**Important: Do not recover an err-disabled port without first identifying the cause.**
If the cause is still present (the bad device is still plugged in), the port will
immediately go err-disabled again.

---

## Procedure - Checking and Verifying pfSense Firewall Rules

When a user on one VLAN reports they cannot reach a resource on another segment,
the pfSense firewall rules are the first place to check.

**In pfSense web UI:**
1. **Firewall > Rules** - select the source VLAN interface tab
2. Review rules from top to bottom - pfSense processes rules in order, first match wins
3. Look for a block rule that matches the traffic in question

**Diagnostics > Packet Capture:**
- Select the source VLAN interface
- Set a filter for the source or destination IP
- Capture traffic and confirm whether packets are arriving at pfSense
- If packets arrive but are blocked: firewall rule is the issue
- If packets do not arrive: the problem is before pfSense (switch, device config)

**Status > System Logs > Firewall:**
- Shows real-time blocked traffic
- Filter by source IP to see what pfSense is blocking from a specific device

---

## Common Switch Commands Reference - Cisco IOS

```bash
# DEVICE INFORMATION
show version                    # IOS version, model, uptime, licence info
show inventory                  # Hardware components and serial numbers

# INTERFACE STATUS
show interfaces status          # All ports: connected/notconnect/err-disabled
show interfaces GigabitEthernet0/X  # Detailed stats for one port (errors, load)
show ip interface brief         # IP addresses on L3 interfaces

# VLAN
show vlan brief                 # All VLANs and their port assignments
show vlan id 10                 # Details for a specific VLAN

# MAC ADDRESS TABLE
show mac address-table          # Full MAC to port mapping
show mac address-table interface GigabitEthernet0/X  # MAC on a specific port

# SPANNING TREE
show spanning-tree              # Full STP topology
show spanning-tree vlan 10      # STP for a specific VLAN
show spanning-tree blockedports # Show which ports STP is blocking

# CDP/LLDP
show cdp neighbors              # Connected Cisco devices with port info
show cdp neighbors detail       # With IP addresses — useful for mapping topology

# LOGGING AND DIAGNOSTICS
show logging                    # System log — error messages and events
show processes cpu              # CPU utilisation
ping 192.168.10.1               # Connectivity test from switch's perspective
```


---