# network-noc-runbook

Structured L1 NOC troubleshooting runbook built from real-world lab experience and CompTIA Network+ curriculum. Covers TCP/IP diagnostics, PRTG alert triage, VLAN architecture, pfSense firewall procedures, VPN troubleshooting, and escalation standards.

---

## Contents

| File | Purpose |
|------|---------|
| `01-connectivity-troubleshooting.md` | OSI-layered internet connectivity triage |
| `02-dns-troubleshooting.md` | DNS failure diagnosis and resolution |
| `03-prtg-alert-response.md` | PRTG sensor DOWN and warning response procedures |
| `04-vpn-troubleshooting.md` | VPN scope check, client-side diagnosis, escalation |
| `05-switch-and-vlan-procedures.md` | Switch port, VLAN assignment, pfSense firewall rules |
| `06-escalation-criteria.md` | ITIL 4 escalation decision tree and handoff standard |
| `diagrams/network-topology.md` | Lab network architecture with ASCII diagram |
| `reference/osi-model-l1-reference.md` | OSI model and common ports for L1 use |

---

## Skills Demonstrated

- **Network Troubleshooting:** OSI-layered methodology, ping, tracert, nslookup, ipconfig, netstat
- **PRTG/Monitoring:** Alert triage, sensor states, threshold interpretation, escalation decisions
- **VLAN / pfSense:** Inter-VLAN routing, firewall rule logic, 802.1Q tagging, DHCP per VLAN
- **VPN:** WireGuard configuration, MTU troubleshooting, client log analysis
- **ITIL 4:** Escalation criteria, P1–P4 priority matrix, documented handoff standard

---

## Lab Environment

Built and tested in a Proxmox-hosted multi-VM environment:
- pfSense 2.7 — VLAN 10/20/99, WireGuard VPN, DHCP per VLAN, firewall rules
- Windows Server 2022 — AD domain controller, DNS, DHCP
- Windows 10/11 clients — domain-joined, VLAN 10
- Ubuntu 22.04 — VLAN 10, Bash scripting lab

---

## Outcome

This runbook reduces NOC first-response time by providing a documented, repeatable triage methodology. The escalation standard ensures L2 receives complete information on handoff — reducing back-and-forth and improving mean time to resolution (MTTR).
```