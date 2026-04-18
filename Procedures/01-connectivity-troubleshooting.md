# Procedure 01 - Internet Connectivity Troubleshooting

**Category:** Network Incident - Connectivity
**ITIL 4 Priority:** P2 (single user) | P1 (multiple users or site-wide)
**Cert alignment:** CompTIA Network+, CompTIA A+
**Average resolution time at L1:** 10–20 minutes (approximately 85% resolved without escalation)
**Last reviewed:** 2026-07

---

## Read Before You Start

Two questions determine everything before you run a single command:

**Question 1: Is this one user/device or multiple?**
If multiple users on the same network segment are affected simultaneously, this is
not a device problem - it is a network infrastructure problem. Skip all steps below
and escalate immediately with the number of affected users and their locations.

**Question 2: When did this start and what changed?**
"It was working this morning" narrows the window. "Just moved my desk" suggests a
cable or switch port issue. "IT updated something yesterday" suggests a config change.
The answer to this question often points directly at the layer to start investigating.

---

## OSI-Layered Troubleshooting Procedure

The OSI model exists precisely for this - it gives you a systematic top-down
(or bottom-up) framework so you do not waste 20 minutes investigating DNS when
the ethernet cable is loose. Always work from Layer 1 upward.

**The rule: never skip a layer. "Ping works but the website doesn't" = Layer 3 is fine,
look at Layer 7 (application). "Ping fails to gateway" = Layer 3 fails, look at Layer 1–2
first before assuming a routing issue.**

---

### Layer 1 - Physical (check this before anything else)

The majority of "internet is down" reports at L1 are physical-layer issues.
Before opening a terminal, look at the hardware.

**Visual checks (30 seconds before any command):**
- NIC indicator lights on the back of the PC: is the link light on (solid) and the activity light blinking when traffic should be flowing?
- Is the ethernet cable firmly seated at BOTH ends - the PC and the switch/wall jack?
- Is the switch port LED showing link status?
- If wireless: is Wi-Fi enabled? Check the Windows tray icon, not just the Fn+F key toggle (some keys toggle the radio off without visual indication)
- If wireless: is the user connected to the correct SSID? A user connecting to a neighbour's network or a guest SSID will have internet on that SSID but no access to corporate resources

**PowerShell NIC check:**
```powershell
# Check all network adapters - look for Status "Up" with a valid link speed
Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription

# If a NIC shows "Disabled":
Enable-NetAdapter -Name "Ethernet"   # Replace with actual adapter name from above

# Check for IP address conflict (two devices with same IP on same segment)
arp -a
# If two different MACs map to the same IP: IP conflict — escalate
```

**Layer 1 passes when:**
- The NIC shows Status: "Up" with a numeric link speed (100 Mbps or 1 Gbps, not "Unknown")
- No error flags in Device Manager for the network adapter
- Physical cable and switch port visually confirmed

---

### Layer 2 - Data Link (VLAN, MAC, switching)

Layer 2 issues are less visible than Layer 1 but equally common in managed-switch environments.

**Checks:**
- Does Windows show "Unidentified Network" or "No network access" despite the NIC being up? → Layer 2 issue (VLAN misconfiguration, 802.1X authentication failure, or switch port error state)
- In a VLAN environment: confirm the switch port is assigned to the correct VLAN for this device

```powershell
# Check the ARP table for duplicate entries (IP conflict at Layer 2)
arp -a

# Check for "Unidentified Network" status
Get-NetConnectionProfile | Select-Object Name, NetworkCategory, IPv4Connectivity
```

---

### Layer 3 - Network / IP Addressing

This is where most triage commands live. Run them in this exact order - each one
tests a specific layer before moving to the next.

```cmd
:: Step 1: Check IP configuration - read carefully before anything else
:: Look for: valid IP (not 169.254.x.x), subnet mask, default gateway, DNS servers
ipconfig /all

:: Step 2: Ping the loopback address - tests the local TCP/IP stack
:: If this fails: TCP/IP stack is corrupt. Go to Winsock Reset below.
ping 127.0.0.1

:: Step 3: Ping the default gateway - tests local network connectivity
:: If this fails: the device cannot reach the local network (cable, DHCP, VLAN issue)
:: Replace with the actual gateway IP from ipconfig /all
ping <default-gateway-ip>

:: Step 4: Ping a public IP address - tests internet routing (bypasses DNS entirely)
:: If this fails but gateway succeeds: routing/WAN issue — escalate
:: If this succeeds: internet routing is working, DNS may be the issue
ping 8.8.8.8

:: Step 5: Ping a hostname - tests DNS resolution
:: If 8.8.8.8 works but this fails: DNS problem only, not internet
ping google.com
```

**Interpretation table - use this every single time:**

| Loopback (127.0.0.1) | Gateway | 8.8.8.8 | google.com | Diagnosis | L1 Action |
|----------------------|---------|---------|------------|-----------|-----------|
| ❌ Fails | — | — | — | TCP/IP stack corrupt | Winsock reset → restart |
| ✅ | ❌ Fails | — | — | Local network issue | Check Layer 1–2; check DHCP |
| ✅ | ✅ | ❌ Fails | — | WAN/routing issue | Escalate — not device-side |
| ✅ | ✅ | ✅ | ❌ Fails | DNS failure only | See Procedure 02 |
| ✅ | ✅ | ✅ | ✅ | Network is fine | Investigate application layer |

---

### DHCP Failure - 169.254.x.x (APIPA) Address

An IP address starting with 169.254 means the device tried to get an IP from DHCP
and received no response. It has assigned itself an APIPA (Automatic Private IP
Addressing) address. A device with an APIPA address cannot communicate outside its
own subnet - it has no valid gateway and no valid DNS.

```cmd
:: Release any existing DHCP lease
ipconfig /release

:: Request a new lease from the DHCP server
ipconfig /renew

:: Verify - check that a valid IP has been assigned (not 169.254.x.x)
ipconfig /all
```

If renewal fails consistently:
- DHCP server may be down or unreachable
- The device may be on the wrong VLAN with no DHCP scope
- The DHCP scope may be exhausted (no addresses available)
All three require escalation.

---

### Layer 4 - Transport (port and protocol reachability)

When a user can reach the internet (8.8.8.8 and google.com both ping) but a specific
application does not work, the issue is often at Layer 4 - a firewall is blocking the
specific port the application uses.

```powershell
# Test whether a specific port is reachable on a remote host
# Example: test RDP (3389) to a server
Test-NetConnection -ComputerName 192.168.1.100 -Port 3389

# Check if the local Windows Firewall is enabled (and potentially blocking)
netsh advfirewall show allprofiles state

# Check active TCP connections and listening ports
netstat -an | findstr ESTABLISHED
netstat -an | findstr LISTENING
```

---

### Winsock Reset (Last Resort - Requires Restart)

Use only after all above steps have failed. This resets the TCP/IP stack to a
clean state. It resolves cases where the stack itself is corrupt (rare but real - often caused by VPN software removal, antivirus interference, or malformed updates).

**Always warn the user that a restart is required before running these commands.**

```cmd
netsh winsock reset
netsh int ip reset
ipconfig /flushdns
ipconfig /release
ipconfig /renew
```

**Restart the computer after running these commands. They do not take effect without a restart.**

---

## Escalation Trigger

Escalate to L2 if any of the following are true:
- Multiple users or devices affected on the same segment (potential P1)
- Default gateway unreachable from multiple devices (switch or uplink failure)
- DHCP renewal fails consistently - scope exhausted or DHCP server down
- Issue persists after winsock reset and full restart
- VLAN misconfiguration suspected (requires switch admin access)

**L2 escalation package must include:**
- Full `ipconfig /all` output (screenshot or copy-paste)
- Results of all four ping tests with actual output shown
- Whether one user or multiple users are affected
- Steps taken in order with the result of each


---