# Useful Network Commands - Windows and Linux Reference

Complete reference for network diagnostic commands used in L1 NOC triage,
covering both Windows Command Prompt / PowerShell and Linux/Bash equivalents.

---

## IP Configuration

| Task | Windows (cmd) | Windows (PowerShell) | Linux (Bash) |
|------|--------------|---------------------|-------------|
| Show IP configuration | `ipconfig /all` | `Get-NetIPConfiguration` | `ip addr show` or `ifconfig` |
| Show routing table | `route print` | `Get-NetRoute` | `ip route` or `route -n` |
| Show ARP table | `arp -a` | `Get-NetNeighbor` | `arp -n` |
| Release DHCP lease | `ipconfig /release` | — | `dhclient -r eth0` |
| Renew DHCP lease | `ipconfig /renew` | — | `dhclient eth0` |
| Flush DNS cache | `ipconfig /flushdns` | `Clear-DnsClientCache` | `sudo systemd-resolve --flush-caches` |

---

## Connectivity Testing

| Task | Windows | Linux |
|------|---------|-------|
| Basic ping | `ping <host>` | `ping -c 4 <host>` |
| Extended ping | `ping <host> -n 100` | `ping -c 100 <host>` |
| Don't-fragment ping (MTU test) | `ping <host> -f -l 1400` | `ping -M do -s 1400 <host>` |
| Trace route | `tracert <host>` | `traceroute <host>` |
| Path ping with loss stats | `pathping <host>` | `mtr <host>` (requires mtr package) |
| Test TCP port | `Test-NetConnection -ComputerName <host> -Port <port>` | `nc -zv <host> <port>` or `telnet <host> <port>` |

---

## DNS

| Task | Windows | Linux |
|------|---------|-------|
| Query DNS for hostname | `nslookup <hostname>` | `nslookup <hostname>` or `dig <hostname>` |
| Query specific DNS server | `nslookup <hostname> <dns-server-ip>` | `dig @<dns-server-ip> <hostname>` |
| Reverse DNS lookup | `nslookup <ip-address>` | `dig -x <ip-address>` |
| Flush DNS cache | `ipconfig /flushdns` | `sudo systemd-resolve --flush-caches` |
| Show DNS cache | `ipconfig /displaydns` | `resolvectl statistics` |

---

## Active Connections and Ports

| Task | Windows | Linux |
|------|---------|-------|
| Show all connections | `netstat -an` | `netstat -an` or `ss -an` |
| Show listening ports | `netstat -an \| findstr LISTENING` | `netstat -tlnp` or `ss -tlnp` |
| Show connections with PIDs | `netstat -ano` | `netstat -anop` or `ss -anop` |
| Match PID to process | `tasklist \| findstr <PID>` | `ps aux \| grep <PID>` |

---

## Wireless (Windows)

```cmd
:: Show available wireless networks
netsh wlan show networks

:: Show current wireless connection details (signal strength, channel, SSID)
netsh wlan show interfaces

:: Disconnect from current SSID
netsh wlan disconnect

:: Connect to a specific SSID (must have a saved profile)
netsh wlan connect name="YourSSID"

:: Show saved wireless profiles
netsh wlan show profiles

:: Export a wireless profile (including password in cleartext)
netsh wlan export profile name="ProfileName" key=clear
```

---

## Linux Network Administration (Ubuntu/Debian)

```bash
# Show all network interfaces and their status
ip addr show
ip link show

# Show routing table
ip route show

# Test connectivity
ping -c 4 8.8.8.8
traceroute google.com
mtr --report google.com        # Better than traceroute — shows packet loss per hop

# DNS queries
dig google.com
dig @8.8.8.8 google.com        # Query a specific DNS server
dig -x 8.8.8.8                 # Reverse DNS lookup

# Check which process is listening on a port
sudo ss -tlnp | grep :443
sudo lsof -i :443

# Restart networking (Ubuntu 18.04+)
sudo netplan apply
sudo systemctl restart NetworkManager

# Show current firewall rules (UFW)
sudo ufw status verbose

# Check interface statistics (errors, dropped packets)
ip -s link show eth0
```


---