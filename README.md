# network-noc-runbook

Structured L1 NOC troubleshooting runbook built from CompTIA Network+ curriculum and
hands-on pfSense, VLAN, and Proxmox lab experience. Covers internet connectivity
triage using an OSI-layered methodology, DNS failure diagnosis, PRTG network monitoring
alert response, VPN troubleshooting, pfSense VLAN architecture and firewall procedures,
and ITIL 4 escalation standards with complete handoff templates.

Built for practical use by L1 NOC engineers - not as a textbook reference, but as a
field guide that a technician can follow in real time during an incident.

---

## What this repo contains and why it matters

| File | Purpose | Who it helps |
|------|---------|--------------|
| `01-connectivity-troubleshooting.md` | OSI-layered internet triage with interpretation table - tells you exactly which layer is failing based on which ping succeeds | Any L1 engineer, every day |
| `02-dns-troubleshooting.md` | DNS failure diagnosis: nslookup, port testing, cache flush, isolation testing - covers the case where "internet works" but "website doesn't load" | L1 triage for one of the most common mis-diagnosed issues |
| `03-prtg-alert-response.md` | PRTG sensor DOWN and WARNING response procedures: scope check, history graph analysis, port testing, escalation package | NOC L1 engineers on monitoring shift |
| `04-vpn-troubleshooting.md` | VPN scope check (P1 vs P2), client log analysis, MTU testing and fixing, wired vs wireless isolation, escalation package | Remote support for work-from-home environments |
| `05-switch-and-vlan-procedures.md` | VLAN assignment verification, pfSense inter-VLAN firewall rules, err-disabled port recovery, switch command reference | MSP engineers, network-aware L1 engineers |
| `06-escalation-criteria.md` | ITIL 4 escalation decision criteria, urgency guide, complete L2 handoff template, ticket closure checklist | All L1 engineers - defines when and how to escalate |
| `diagrams/network-topology.md` | Full lab architecture with ASCII diagram, VLAN table, WireGuard config, firewall rule summary, and design decision justifications | Any engineer onboarding to the lab or reviewing the architecture |
| `reference/common-ports-and-protocols.md` | 30+ ports with service names, direction, and notes - plus PowerShell port testing snippets | Quick reference during firewall troubleshooting |
| `reference/osi-model-quick-reference.md` | OSI layers mapped to L1 diagnostic tools and actions, plus symptom-to-layer matching table | New L1 engineers learning the structured troubleshooting approach |
| `reference/useful-network-commands.md` | Complete Windows and Linux network command reference with side-by-side equivalents | Cross-platform support engineers |

---

## Skills demonstrated

**Network troubleshooting:**
OSI-layered diagnostic methodology, ping/tracert/pathping/nslookup/ipconfig,
DHCP failure diagnosis, DNS failure diagnosis, Winsock stack repair, MTU testing,
Layer 1-4 fault isolation

**PRTG monitoring:**
Alert state interpretation, sensor type understanding, scope assessment, 24-hour history
graph analysis, threshold reference, escalation procedures

**pfSense and VLAN:**
Inter-VLAN routing design, firewall rule logic (first-match, source-interface rules),
802.1Q VLAN tagging, DHCP per VLAN, IoT isolation, management plane protection

**VPN:**
Scope assessment (single vs multi-user), client log analysis by VPN type, MTU
fragmentation testing and remediation, wired vs wireless isolation, escalation packaging

**ITIL 4:**
P1-P4 priority matrix, escalation decision criteria, escalation urgency guide,
complete L2 handoff template, ticket closure checklist, Incident vs Problem vs Change

---

## Lab environment

```
Hypervisor  : Proxmox VE 8.x
Firewall    : pfSense 2.7 (VM) — VLAN 10/20/99, WireGuard VPN, DHCP per VLAN
DC          : Windows Server 2022 — AD DS, DNS, DHCP
Clients     : Windows 10 22H2, Windows 11 23H2 — domain-joined on VLAN 10
Ubuntu      : Ubuntu 22.04 LTS — VLAN 10, Bash scripting, Splunk Forwarder
Kali        : Kali Linux 2024 — VLAN 20 (isolated), pen testing
Splunk      : Splunk Free on Ubuntu — VLAN 99 (management)
```

---

## Impact

This runbook reduces NOC first-response time by providing a documented, repeatable
triage methodology that a new L1 engineer can follow on day one without needing
to rely on tribal knowledge or asking a senior colleague for every incident.

The escalation standard ensures L2 engineers receive complete, structured information
on first contact - eliminating the back-and-forth that typically consumes 40–60% of
P2 incident resolution time.

The OSI-layered approach prevents the most common L1 diagnostic error: investigating
the wrong layer. Treating a DNS failure (Layer 7) as a routing problem (Layer 3), or
a cable issue (Layer 1) as a DHCP configuration error (Layer 3), both waste time and
erode user confidence. The interpretation table in Procedure 01 eliminates these errors.