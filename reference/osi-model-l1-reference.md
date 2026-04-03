# OSI Model — L1 NOC Quick Reference

## Layers and L1 Relevance

| Layer | Name | L1 Relevance | Tools |
|-------|------|-------------|-------|
| 1 — Physical | Cables, NICs, switches | Check first. Always. | Visual inspection, cable tester |
| 2 — Data Link | MAC addresses, VLANs, switching | ARP, MAC table, VLAN assignment | arp -a, show mac address-table |
| 3 — Network | IP addresses, routing | IP config, ping, routing table | ipconfig, ping, tracert, route print |
| 4 — Transport | TCP/UDP, ports | Port reachability, connection state | netstat, Test-NetConnection |
| 5–7 | Session/Presentation/Application | Usually L2+ scope | Application logs |