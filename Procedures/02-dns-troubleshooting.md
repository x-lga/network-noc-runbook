# Procedure 02 - DNS Troubleshooting

**Category:** Network / Name Resolution  
**ITIL 4 Priority:** P2 (user cannot reach resources by name)  

---

## Symptoms

- Can ping 8.8.8.8 but cannot ping google.com
- "Server not found" in browser
- Applications that use hostnames fail; applications using IP work

---

## Diagnostic Steps

### Step 1 - Check configured DNS servers

```cmd
ipconfig /all
```

Look for: `DNS Servers . . . . . . . . . . : 192.168.1.1`

If blank or shows 169.254.x.x - DHCP failure. See Procedure 01.

---

### Step 2 - Test DNS resolution with nslookup

```cmd
# Query your internal DNS server
nslookup google.com

# Query a known public DNS server directly (bypasses internal DNS)
nslookup google.com 8.8.8.8

# Query an internal resource
nslookup dc01.contoso.local
```

**Interpretation:**

| nslookup (internal) | nslookup (8.8.8.8) | Cause |
|---------------------|-------------------|-------|
| Fails | Passes | Internal DNS server issue - escalate |
| Fails | Fails | Firewall blocking DNS (port 53 UDP/TCP) |
| Passes | — | DNS cache stale - flush and retry |

---

### Step 3 - Flush DNS cache

```cmd
ipconfig /flushdns
```

Test again with nslookup and browser after flush.

---

### Step 4 - Check DNS port reachability

```powershell
# Test if DNS port 53 is reachable on the internal DNS server
Test-NetConnection -ComputerName 192.168.1.1 -Port 53
```

---

### Step 5 - Change DNS to public (temporary test only)

```powershell
# Temporarily change DNS to 8.8.8.8 to confirm internal DNS is the problem
# ALWAYS revert after testing — never leave public DNS in a corporate environment
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "8.8.8.8"

# Revert:
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses
```

---

## Escalation Trigger

- Internal DNS server unreachable → L2 (DNS server issue, not client)
- Port 53 blocked → L2 (firewall rule change required)
- Domain controller DNS failing for multiple users → P1 escalation
```

---