# Procedure 01 — Connectivity Troubleshooting

**Category:** Network Incident  
**ITIL 4 Priority:** P2 (single user) / P1 (multiple users or site-wide)  

---

## Symptoms

- User reports "no internet" or "cannot reach network resources"
- Browser shows "ERR_NAME_NOT_RESOLVED" or "This site can't be reached"
- Mapped drives disconnected

---

## Layered Troubleshooting Procedure (OSI Model Approach)

Work from Layer 1 upward. Do not skip layers.

### Layer 1 — Physical

- [ ] Check NIC lights on the back of the PC: link light (solid) + activity light (blinking)
- [ ] Reseat ethernet cable at both ends
- [ ] Try a different ethernet cable
- [ ] If wireless: check if Wi-Fi is enabled (Windows: taskbar Wi-Fi icon)
- [ ] If wireless: ensure the user is connected to the correct SSID (not a neighbour's network)

**If Layer 1 passes:** Proceed to Layer 2

---

### Layer 2 — Data Link

```powershell
# Check NIC status
Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MacAddress

# Check if NIC is enabled
Enable-NetAdapter -Name "Ethernet"   # Replace with actual adapter name
```

- [ ] NIC shows "Up" status and a valid link speed
- [ ] No "Unidentified Network" warnings in Network Connections

---

### Layer 3 — Network (IP / Routing)

```cmd
# Check IP configuration — look for valid IP, subnet, gateway, DNS
ipconfig /all

# Ping loopback — confirms TCP/IP stack is functional
ping 127.0.0.1

# Ping default gateway — confirms local network connectivity
ping <default-gateway-ip>

# Ping external IP — confirms internet routing (bypasses DNS)
ping 8.8.8.8

# Ping external hostname — tests DNS resolution
ping google.com
```

**Interpretation table:**

| Loopback | Gateway | 8.8.8.8 | google.com | Likely Cause |
|----------|---------|---------|------------|-------------|
| Fail | — | — | — | TCP/IP stack corrupt → netsh winsock reset |
| Pass | Fail | — | — | Local network issue (cable, VLAN, switch port) |
| Pass | Pass | Fail | — | Router/WAN issue → check gateway, escalate |
| Pass | Pass | Pass | Fail | DNS failure → see Procedure 02 |
| Pass | Pass | Pass | Pass | Issue is application-level → not network |

---

### Layer 4 — Transport

```cmd
# Check if a specific port is reachable (example: RDP port 3389)
Test-NetConnection -ComputerName 192.168.1.100 -Port 3389

# Check local firewall is not blocking
netsh advfirewall show allprofiles state
```

---

## DHCP Release and Renew

If the device has an APIPA address (169.254.x.x), DHCP has failed:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

---

## Winsock Reset (Last Resort Before Escalation)

```cmd
netsh winsock reset
netsh int ip reset
ipconfig /flushdns
ipconfig /release
ipconfig /renew
```

**Always restart after winsock reset.**

---

## Escalation Trigger

Escalate to L2 if:
- Gateway unreachable from multiple devices on same switch
- IP address cannot be obtained after /release /renew
- Issue persists after winsock reset and restart

**Escalation documentation must include:**
- ipconfig /all output (screenshot or paste)
- Results of each ping command
- Steps taken and results


