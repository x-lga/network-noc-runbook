# OSI Model Quick Reference - L1 NOC Field Guide

The OSI model exists to give you a structured language for diagnosing network
problems. When a user says "the internet is down," the OSI model tells you
where to start looking and what to check at each layer.

---

## The Seven Layers - L1 Relevance and Diagnostic Actions

| Layer | Number | Name | What it handles | L1 diagnostic tool/action |
|-------|--------|------|----------------|--------------------------|
| Physical | 1 | Wires, signals, hardware | Electrical signals, cables, NIC hardware, switch ports | Visual inspection; NIC lights; cable swap; `Get-NetAdapter` |
| Data Link | 2 | Frames, MAC addresses, VLANs | Ethernet frames, MAC addressing, 802.1Q VLAN tagging, switching | `arp -a`; VLAN check; switch port status; `Get-NetConnectionProfile` |
| Network | 3 | Packets, IP addressing, routing | IP addresses, subnet masks, routing between networks | `ipconfig /all`; `ping`; `tracert`; `route print` |
| Transport | 4 | Segments, TCP/UDP, ports | End-to-end connections, port numbers, reliability (TCP) vs speed (UDP) | `netstat`; `Test-NetConnection -Port`; firewall rules |
| Session | 5 | Sessions | Establishing, maintaining, and terminating sessions | Typically L2+ - session establishment errors in application logs |
| Presentation | 6 | Data format, encryption | Data encoding, compression, SSL/TLS | Certificate errors; TLS version mismatch - L2+ scope |
| Application | 7 | End-user protocols | HTTP, FTP, DNS, SMTP, RDP - what the application uses | Application logs; browser dev tools; HTTP status codes |

---

## The L1 Troubleshooting Rule

**Always start at Layer 1 and work upward. Never skip a layer.**

A DNS failure (Layer 7) looks identical to a routing failure (Layer 3) from the
user's perspective - "I can't access the website." The OSI model tells you to:
1. Confirm Layer 1 is working (NIC lights, cable)
2. Confirm Layer 3 is working (ping 127.0.0.1, ping gateway, ping 8.8.8.8)
3. Then test Layer 7 (ping google.com - tests DNS)

Skipping straight to DNS (Layer 7) when the cable is loose (Layer 1) wastes
diagnostic time on the wrong layer.

---

## Practical Layer Matching - Common Symptoms

| Symptom | Layer | Likely Cause | First Action |
|---------|-------|-------------|-------------|
| NIC shows "No network" in OS | 1 – 2 | Cable, NIC, switch port | Visual inspection, cable swap |
| "Unidentified Network" in Windows | 2 – 3 | DHCP failure or VLAN mismatch | `ipconfig /all`; check for 169.254.x.x |
| Can ping gateway, cannot ping 8.8.8.8 | 3 | WAN/routing issue | Escalate - not device-side |
| Can ping 8.8.8.8, cannot ping google.com | 3 – 7 | DNS failure | `ipconfig /flushdns`; `nslookup google.com` |
| Website loads, specific app doesn't | 4 – 7 | Port blocked by firewall | `Test-NetConnection -Port`; check firewall rules |
| RDP fails to specific server | 4 | Port 3389 blocked or RDP disabled | `Test-NetConnection -ComputerName X -Port 3389` |
| Slow internet for one user | 1 – 4 | Duplex mismatch, congested path | `pathping`; check switch port stats for errors |
| Slow internet for everyone | 3 – 4 | WAN saturation or ISP issue | `tracert` to identify where latency enters; escalate |

---

## Network Diagnostic Commands - Quick Reference

```cmd
:: IP configuration — the first command to run in any network issue
ipconfig /all

:: Release and renew DHCP lease
ipconfig /release
ipconfig /renew

:: Flush DNS resolver cache
ipconfig /flushdns

:: Show DNS cache contents
ipconfig /displaydns

:: ARP table — MAC to IP mappings on local segment
arp -a

:: Ping tests (see Procedure 01 for interpretation)
ping 127.0.0.1
ping <gateway-ip>
ping 8.8.8.8
ping google.com

:: Trace route — shows each hop and latency
tracert google.com

:: Pathping — combines tracert with sustained loss measurement
pathping google.com

:: DNS query tool
nslookup google.com
nslookup google.com 8.8.8.8
nslookup dc01.contoso.local

:: Active connections and listening ports
netstat -an
netstat -ano

:: Windows Firewall status
netsh advfirewall show allprofiles state

:: Network stack reset (requires restart)
netsh winsock reset
netsh int ip reset
```

```powershell
:: PowerShell network diagnostics
Get-NetAdapter | Select-Object Name, Status, LinkSpeed
Test-NetConnection -ComputerName 8.8.8.8
Test-NetConnection -ComputerName 192.168.1.100 -Port 443
Get-DnsClientServerAddress
Resolve-DnsName google.com
```

---