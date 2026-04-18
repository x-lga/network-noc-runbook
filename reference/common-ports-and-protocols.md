# Common Ports and Protocols - L1 NOC Quick Reference

Used for: port-level connectivity testing, firewall rule verification,
interpreting netstat output, and understanding which service a blocked connection affects.

---

## Essential Ports - L1 Must Know

| Port | Protocol | Service | Direction (typical) | Notes |
|------|---------|---------|-------------------|-------|
| 20, 21 | TCP | FTP (data, control) | Outbound | Legacy - prefer SFTP. Often blocked. |
| 22 | TCP | SSH | Outbound | Secure remote terminal to Linux/Unix systems |
| 23 | TCP | Telnet | Outbound | Unencrypted - should not be in use in any modern environment |
| 25 | TCP | SMTP | Inbound to mail server | Email delivery - direct SMTP often blocked by ISPs |
| 53 | UDP / TCP | DNS | Both | DNS queries use UDP; large responses and zone transfers use TCP |
| 67, 68 | UDP | DHCP | Server 67, Client 68 | Client broadcasts on 68, server responds on 67 |
| 80 | TCP | HTTP | Outbound | Unencrypted web traffic - should redirect to HTTPS |
| 110 | TCP | POP3 | Outbound | Legacy email client retrieval - largely replaced by IMAP |
| 123 | UDP | NTP | Outbound | Network Time Protocol - domain computers sync time to DC |
| 143 | TCP | IMAP | Outbound | Email client retrieval - still in use |
| 161, 162 | UDP | SNMP | 161 queries, 162 traps | Network device monitoring - PRTG uses SNMP |
| 389 | TCP | LDAP | Both | Active Directory queries - unencrypted |
| 443 | TCP | HTTPS | Outbound | Encrypted web traffic - majority of internet traffic |
| 445 | TCP | SMB | Both | Windows file sharing, AD replication, mapped drives |
| 465 | TCP | SMTPS | Outbound | Encrypted SMTP (SSL) - email sending from clients |
| 587 | TCP | SMTP (STARTTLS) | Outbound | Preferred encrypted email submission port |
| 636 | TCP | LDAPS | Both | Encrypted LDAP - secure AD queries |
| 993 | TCP | IMAPS | Outbound | Encrypted IMAP |
| 995 | TCP | POP3S | Outbound | Encrypted POP3 |
| 1194 | UDP | OpenVPN | Both | OpenVPN default - may use TCP 443 as fallback |
| 3389 | TCP | RDP | Inbound to server | Remote Desktop Protocol - target for brute force |
| 3306 | TCP | MySQL | Both | Database - should not be exposed externally |
| 5060 | UDP/TCP | SIP | Both | VoIP signalling |
| 5985 | TCP | WinRM (HTTP) | Both | PowerShell remoting - unencrypted |
| 5986 | TCP | WinRM (HTTPS) | Both | PowerShell remoting - encrypted |
| 8080 | TCP | HTTP Alt | Outbound | Alternative HTTP - web proxies, dev servers |
| 8443 | TCP | HTTPS Alt | Outbound | Alternative HTTPS - common for admin web interfaces |
| 51820 | UDP | WireGuard | Both | WireGuard VPN - lab configuration |

---

## Testing Port Connectivity

```powershell
# Test if a TCP port is reachable on a remote host
Test-NetConnection -ComputerName 192.168.1.100 -Port 443

# Quick port scan of common ports on a target
$target = "192.168.1.100"
$ports  = @(22, 53, 80, 443, 3389, 5985)
foreach ($port in $ports) {
    $result = Test-NetConnection -ComputerName $target -Port $port -WarningAction SilentlyContinue
    $status = if ($result.TcpTestSucceeded) { "OPEN" } else { "CLOSED/FILTERED" }
    Write-Host "Port $port : $status"
}
```

```cmd
:: Test a port using netcat equivalent (built into Windows 10+)
Test-NetConnection -ComputerName <host> -Port <port>
```

---

## Checking Active Connections on a Local Machine

```cmd
:: Show all active TCP connections with process IDs
netstat -ano

:: Show all listening ports
netstat -an | findstr LISTENING

:: Match a PID to a process name
tasklist | findstr <PID>

:: All-in-one: show connections with process names (requires admin)
netstat -b
```


---