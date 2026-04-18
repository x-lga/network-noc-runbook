# Procedure 04 - VPN Troubleshooting

**Category:** Remote Access
**ITIL 4 Priority:** P2 (single user) | P1 (multiple users)
**Cert alignment:** CompTIA Network+, CompTIA Security+
**Average resolution time at L1:** 15–30 minutes (single user) | Immediate escalation (multiple users)
**Last reviewed:** 2026-07

---

## The Single Most Important First Step: Scope Check

Before running any diagnostic command, ask one question:
**"Are other remote users also having VPN problems right now?"**

This determines whether you spend 30 minutes on client-side troubleshooting that
will ultimately be irrelevant, or you escalate a P1 within 5 minutes.

| Scope | Classification | L1 Action |
|-------|---------------|-----------|
| Multiple users cannot connect | P1 - Server-side issue | Escalate IMMEDIATELY. Phone call, not just a ticket. |
| Single user cannot connect | P2 - Client-side investigation | Continue with the steps below |

**For P1 multi-user VPN failure:** Check the VPN server status in your monitoring
platform, notify L2 by phone, and document the number of affected users and the
time the issue started. Do not spend time on individual client steps.

---

## Step 1 - Confirm Local Internet is Working

VPN cannot connect if the local internet connection is broken.
This sounds obvious but is frequently skipped, wasting significant diagnostic time.

```cmd
:: Confirm basic internet connectivity before troubleshooting VPN
ping 8.8.8.8

:: If this fails: local internet is the problem, not VPN
:: Go to Procedure 01 (connectivity troubleshooting) first
```

---

## Step 2 - Check VPN Client Version

An outdated VPN client is a common cause of connection failures, especially after
a server-side certificate renewal or protocol update.

```powershell
:: Check installed software versions (PowerShell — works for any installed app)
Get-WmiObject -Class Win32_Product |
    Where-Object { $_.Name -like "*VPN*" -or $_.Name -like "*AnyConnect*" -or $_.Name -like "*GlobalProtect*" } |
    Select-Object Name, Version
```

---

## Step 3 - Review VPN Client Logs

VPN client logs contain the exact reason for disconnection or failure. Reading the
log before trying anything else saves significant diagnostic time.

**Log file locations by client:**

| VPN Client | Log Location |
|-----------|-------------|
| Cisco AnyConnect | `%ProgramData%\Cisco\Cisco AnyConnect Secure Mobility Client\temp\AnyConnect.log` |
| Palo Alto GlobalProtect | `%APPDATA%\Palo Alto Networks\GlobalProtect\PanGPS.log` |
| OpenVPN | `C:\Program Files\OpenVPN\log\` |
| WireGuard | WireGuard app → click the tunnel name → Show Log |
| Pulse Secure | `%APPDATA%\Pulse Secure\Logs\` |
| FortiClient | `C:\Users\[user]\AppData\Roaming\Fortinet\FortiClient\logs\` |

**What to look for in the log:**
- Authentication failure messages (wrong credentials, certificate error, expired token)
- Connection refused messages (server unreachable or down)
- Protocol or cipher mismatch errors (client version vs server mismatch)
- Certificate errors (server certificate expired or untrusted)
- Disconnect reason codes (each VPN client has its own code format)

---

## Step 4 - MTU Test and Fix

VPN encapsulates packets in an additional header, reducing the effective MTU
(Maximum Transmission Unit). If the MTU is set too high, large packets get
fragmented — and some networks drop fragmented VPN packets, causing intermittent
drops or complete connection failures.

This is a common cause of "VPN connects but disconnects randomly" reports.

```cmd
:: Test whether large packets are being fragmented
:: The -f flag sets the "do not fragment" bit
:: Start at 1400 and decrease until ping succeeds
ping 8.8.8.8 -f -l 1400
ping 8.8.8.8 -f -l 1300
ping 8.8.8.8 -f -l 1200

:: If 1400 fails but 1300 succeeds: MTU issue confirmed
:: The working size + 28 (IP+ICMP header overhead) = maximum MTU
```

**Fix: set MTU on the Wi-Fi adapter:**
```cmd
:: Set MTU to 1400 on the Wi-Fi adapter (persistent across reboots)
netsh interface ipv4 set subinterface "Wi-Fi" mtu=1400 store=persistent

:: Verify the change
netsh interface ipv4 show subinterface
```

---

## Step 5 - Wired vs Wireless Test

Wi-Fi instability is one of the most common causes of VPN drops.
VPN connections are highly sensitive to packet loss - even 1-2% packet loss
on a Wi-Fi connection can cause a VPN to disconnect.

Ask the user:
1. **Plug in an ethernet cable** (or connect to a mobile hotspot if ethernet is unavailable)
2. Retry the VPN connection
3. If VPN is stable on wired/hotspot but drops on Wi-Fi: the Wi-Fi connection is the problem

**If Wi-Fi is the confirmed cause:**
- Check Wi-Fi signal strength (should be > -70 dBm for stable VPN)
- Update Wi-Fi adapter driver: Device Manager → Network Adapters → Wi-Fi → Update Driver
- Try a different wireless frequency band (5 GHz is less congested than 2.4 GHz)
- Move closer to the access point, or use a USB Wi-Fi adapter as a test

---

## Step 6 - Reinstall VPN Client

If all above steps fail, the VPN client installation may be corrupt.

```powershell
:: Check what is installed
Get-WmiObject -Class Win32_Product | Where-Object { $_.Name -like "*VPN*" }

:: Uninstall (replace GUID with the one from above output)
:: Or uninstall through Settings > Apps > Apps & Features
```

1. Uninstall the VPN client fully (Settings > Apps)
2. Restart the computer
3. Download the latest version from your organisation's software portal
4. Install fresh and test

---

## Step 7 - Escalation Package

Escalate to L2 with all of the following:

```
VPN ESCALATION PACKAGE
─────────────────────────────────────────────
User          : [Name and username]
Computer      : [Computer name and OS]
VPN Client    : [Client name and exact version]
─────────────────────────────────────────────
Scope check   : Single user confirmed
Internet      : Working (ping 8.8.8.8 successful)
Log file      : [Attach log file or paste key error messages]
Key log error : [Exact error message or disconnect reason code]
─────────────────────────────────────────────
MTU test      : [Largest size that succeeded / failed at all sizes]
Wired test    : [Stable on wired / also drops on wired / not tested]
Reinstall     : [Performed / not performed]
─────────────────────────────────────────────
When started  : [When did this issue begin?]
Last worked   : [When did it last work correctly?]
Recent changes: [Any Windows updates, software installs, or config changes?]
Business impact: [What cannot the user do right now?]
```


---