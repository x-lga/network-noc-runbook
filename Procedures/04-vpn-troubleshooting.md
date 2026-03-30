# Procedure 04 - VPN Troubleshooting

**Category:** Remote Access  
**ITIL 4 Priority:** P2 (single user) / P1 (multiple users)

---

## Symptoms

- User cannot connect to VPN
- VPN connects but disconnects frequently
- VPN connected but cannot reach internal resources

---

## Step 1 - Scope Check (Critical First Step)

```
Q: Is this one user or multiple users?

Multiple users → P1 escalation immediately. Server-side issue.
    - Notify L2/L3 immediately
    - Check VPN server status in monitoring dashboard
    - Do not spend time on client-side steps

One user → continue with client-side procedure below
```

---

## Step 2 - Local Connectivity Check

```cmd
# Confirm local internet works before blaming VPN
ping 8.8.8.8

# If ping fails: local internet issue — see Procedure 01 first
```

---

## Step 3 - Review VPN Client Logs

VPN client log locations:

| Client | Log Location |
|--------|-------------|
| Cisco AnyConnect | %ProgramData%\Cisco\Cisco AnyConnect Secure Mobility Client\temp\AnyConnect.log |
| GlobalProtect | %APPDATA%\Palo Alto Networks\GlobalProtect\PanGPS.log |
| OpenVPN | C:\Program Files\OpenVPN\log\ |
| WireGuard | Viewable in WireGuard app: Show Log |

Look for: disconnect reason codes, authentication errors, timeout messages.

---

## Step 4 - MTU Check and Fix

VPN encapsulation adds overhead. MTU too high = packet fragmentation = drops.

```cmd
# Test MTU - increase size until ping fails, then subtract 28 for VPN overhead
ping 8.8.8.8 -f -l 1400
ping 8.8.8.8 -f -l 1300

# If 1400 fails but 1300 passes: MTU issue confirmed
# Fix via adapter settings:
netsh interface ipv4 set subinterface "Wi-Fi" mtu=1400 store=persistent
```

---
## Step 5 - WiFi vs Wired Test

Ask user to connect via ethernet cable and retry VPN. If VPN is stable on wired:
- Issue is WiFi instability (interference, weak signal, driver)
- Recommend: update WiFi drivers, move closer to AP, or use wired

---

## Step 6 - Escalation Package

Provide to L2:
- VPN client name and version
- VPN log file (attached)
- Result of scope check (one user or many)
- Local internet connectivity status
- MTU test result
- WiFi vs wired test result
- Time and duration of disconnections


---