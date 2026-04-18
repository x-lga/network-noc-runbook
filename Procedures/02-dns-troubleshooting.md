# Procedure 02 - DNS Troubleshooting

**Category:** Network Incident - Name Resolution
**ITIL 4 Priority:** P2 (single user cannot reach resources by name)
**Cert alignment:** CompTIA Network+
**Average resolution time at L1:** 5–15 minutes
**Last reviewed:** 2026-07

---

## When to Use This Procedure

Use this procedure when:
- Ping to `8.8.8.8` succeeds (internet routing is working)
- Ping to `google.com` fails (name cannot be resolved)
- Browser shows "ERR_NAME_NOT_RESOLVED" or "Server not found"
- Applications that use IP addresses work; applications using hostnames do not

If ping to `8.8.8.8` also fails: the issue is not DNS. Go back to
Procedure 01 (connectivity troubleshooting) - DNS requires working internet routing.

---

## Step 1 - Check Configured DNS Servers

```cmd
ipconfig /all
```

Look for the "DNS Servers" line. It should show your internal DNS server IP
(typically the domain controller - e.g., 192.168.10.10) or your router's IP.

**Problem indicators:**
- DNS Servers line is blank → DHCP is not providing DNS server information
- DNS Servers shows `169.254.x.x` → APIPA address, DHCP failure, see Procedure 01
- DNS Servers shows only `0.0.0.0` → DNS not configured, check DHCP scope options

---

## Step 2 - Test DNS Resolution with nslookup

`nslookup` is the primary DNS diagnostic tool. It queries DNS directly and shows
you exactly what the DNS server returns - or why it cannot.

```cmd
:: Basic test - query your configured DNS server for google.com
nslookup google.com

:: Query a specific DNS server (bypasses your configured one)
:: Use this to isolate whether your internal DNS server is the problem
nslookup google.com 8.8.8.8

:: Query for an internal hostname (must work in a domain environment)
nslookup dc01.contoso.local

:: Query for your domain controller by IP (reverse lookup)
nslookup 192.168.10.10
```

**Interpretation:**

| nslookup internal DNS | nslookup 8.8.8.8 | Diagnosis | Action |
|-----------------------|------------------|-----------|--------|
| ✅ Returns address | — | DNS is working | Issue is application-layer, not DNS |
| ❌ Timeout / SERVFAIL | ✅ Returns address | Internal DNS server problem | Escalate — DNS server issue |
| ❌ Fails | ❌ Fails | Firewall blocking DNS port 53 | Check firewall — see Step 4 |
| Returns stale/wrong IP | — | DNS cache stale | Flush DNS cache — see Step 3 |

---

## Step 3 - Flush the DNS Cache

The DNS resolver cache on Windows stores previously resolved hostnames. If a cached
entry is stale (pointing to an old IP after a server migration) or corrupt, it causes
resolution failures even when the DNS server is working correctly.

```cmd
:: Flush the local DNS resolver cache
ipconfig /flushdns

:: Confirm the cache has been cleared
ipconfig /displaydns
:: Should return empty or very few entries after flush
```

After flushing, retry the original test (`ping google.com` or the failing application).

---

## Step 4 - Test DNS Port Reachability

DNS uses UDP port 53 for standard queries (and TCP port 53 for large responses and
zone transfers). If a firewall is blocking port 53, DNS will fail even when the DNS
server is running.

```powershell
:: Test if DNS port 53 is reachable on the internal DNS server
:: Replace 192.168.10.10 with your actual DNS server IP
Test-NetConnection -ComputerName 192.168.10.10 -Port 53

:: Expected result: TcpTestSucceeded = True
:: If False: firewall rule is blocking DNS — escalate
```

---

## Step 5 - Temporary DNS Change for Isolation (Test Only)

**Only use this step to confirm the internal DNS server is the problem.**
**Always revert after testing. Never leave a corporate machine pointing to public DNS.**

In a corporate environment, machines must use the internal DNS server for:
- Domain name resolution (contoso.local)
- Authentication (Kerberos relies on DNS)
- GPO application
- SCCM/Intune connectivity

Leaving a machine on public DNS after testing will break domain functionality.

```powershell
:: Check current DNS settings
Get-DnsClientServerAddress

:: Temporarily change to Google DNS for testing only
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "8.8.8.8"

:: Test
ping google.com
nslookup google.com

:: REVERT IMMEDIATELY after confirming the test result
:: Restore DHCP-assigned DNS (gets DNS from DHCP server again)
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses

:: Or restore manually to your internal DNS server
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.10.10"
```

---

## Step 6 - Register DNS (Force Dynamic Update)

In a domain environment, if a machine's DNS record is stale or missing (other machines
cannot find it by name), force a dynamic DNS registration:

```cmd
:: Re-register this machine's DNS record with the DNS server
ipconfig /registerdns

:: Verify registration succeeded (may take 30–60 seconds)
nslookup <this-computer-name>.contoso.local
```

---

## Escalation Trigger

Escalate to L2 if:
- Internal DNS server unreachable (port 53 blocked or server down)
- Multiple clients experiencing the same DNS failure simultaneously
- Flushing DNS and using public DNS both fail
- Domain authentication failures accompanying DNS failures (SERVFAIL on internal queries)

DNS server failures are P1 in a domain environment - Kerberos authentication,
GPO application, and all Active Directory operations depend on DNS.


---
