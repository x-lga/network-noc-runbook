# Procedure 03 - PRTG Network Monitor Alert Response

**Category:** NOC Event Management
**ITIL 4 Priority:** Determined by sensor type, device criticality, and number of affected sensors
**Cert alignment:** CompTIA Network+, CompTIA A+
**Last reviewed:** 2026-07

---

## Understanding PRTG Alert States

PRTG uses five status states. Understanding what each means determines your first action.

| Status | Indicator | Meaning | First Action |
|--------|-----------|---------|--------------|
| Up | 🟢 Green | All sensors operating within defined thresholds | No action required - monitor |
| Warning | 🟡 Yellow | Sensor approaching threshold - trend is worsening | Log the time, check the history graph, monitor every 15 minutes |
| Down | 🔴 Red | Device unreachable or sensor threshold breached | Follow the DOWN procedure below |
| Paused | 🔵 Blue | Manually paused by an admin | Verify this is intentional - check the change calendar and who paused it |
| Unknown | ⚫ Grey | No data received from the sensor | Check sensor configuration and whether the PRTG probe can reach the device |

---

## What PRTG Monitors and Why It Matters

Understanding what a sensor measures helps you interpret an alert correctly.

| Sensor Type | What It Measures | Alert Significance |
|-------------|-----------------|-------------------|
| Ping sensor | Round-trip latency and packet loss to a device | DOWN = device unreachable; WARNING = latency spike or packet loss |
| CPU Load sensor | Processor utilisation percentage | WARNING/DOWN = sustained high load - service degradation likely |
| Memory sensor | RAM utilisation percentage | WARNING = approaching saturation; DOWN = critical - application may crash |
| Disk Free Space sensor | Available disk space percentage | WARNING = approaching full; DOWN = below critical threshold |
| HTTP sensor | HTTP response code and response time | DOWN = web server not responding or returning errors |
| SNMP Traffic sensor | Bandwidth utilisation on a network interface | WARNING = approaching capacity; DOWN = interface saturated or down |
| Windows Service sensor | Whether a specific Windows service is running | DOWN = service has stopped - application-level impact |
| Port sensor | Whether a specific TCP port is reachable | DOWN = service not listening or firewall blocking |

---

## Procedure - Sensor Shows DOWN

**Before taking any action, ask: Is this one sensor or many?**

If multiple sensors on the same device or the same network segment are DOWN
simultaneously, this is an infrastructure problem (switch down, power failure,
WAN outage). Skip individual device steps and escalate immediately.

---

### Step 1 - Verify the alert is real

Not every DOWN alert represents an actual outage. A transient 30-second network
blip, a brief maintenance window, or a misconfigured sensor threshold can trigger DOWN.

```cmd
:: Ping the device directly from your NOC workstation
:: Use 20 packets (not just 4) to catch intermittent packet loss
ping <device-ip> -n 20

:: If ping responds consistently:
::   PRTG sensor may be misconfigured (wrong SNMP community string, wrong port,
::   wrong protocol version). The device is up — the sensor is wrong.
::   Note this for the escalation if relevant.

:: If ping fails entirely:
::   Device is genuinely unreachable. Proceed to Step 2.

:: If ping shows packet loss (some replies, some timeouts):
::   Intermittent connectivity - note the loss percentage and continue investigation
```

---

### Step 2 - Check scope - adjacent devices

This single step determines whether you have one failed device or an infrastructure failure.

In the PRTG device tree:
1. Expand the network group or location that contains the DOWN device
2. Look at adjacent devices - switches, routers, servers in the same area
3. If multiple devices are DOWN: **the problem is the infrastructure they share** - the
   upstream switch, the distribution switch, the WAN connection, or the power feed

**If multiple devices down: Do not spend time on individual device steps.**
Escalate immediately with:
- List of all DOWN devices
- Their location or network segment
- The time the first alert fired
- Whether adjacent devices share a common switch, power circuit, or WAN link

---

### Step 3 - Review the 24-hour history graph

Click the DOWN sensor → Graph tab → change timeframe to Last 24 hours.

What you see tells you the failure mode:

| Pattern | What it indicates |
|---------|-------------------|
| Clean drop to zero at a specific time | Sudden failure - link cut, power loss, device crash |
| Gradual degradation before the drop | Resource exhaustion, memory leak, hardware wear |
| Up/down/up/down pattern (flapping) | Unstable link - cable, SFP module, duplex mismatch |
| DOWN at the same time every day | Scheduled job or backup consuming resources; check change calendar |
| Recently installed sensor with immediate DOWN | Sensor misconfiguration - wrong IP, SNMP community, or protocol |

Document what you observe. This context accelerates L2 diagnosis significantly.

---

### Step 4 - Test specific service availability

For sensors that monitor a service (HTTP, port, Windows Service), the device may be
up but the specific service may be down.

```powershell
:: Test a specific TCP port (e.g., port 80 for HTTP, 443 for HTTPS, 3389 for RDP)
Test-NetConnection -ComputerName <device-ip> -Port 80

:: Quick HTTP check using PowerShell
try {
    $response = Invoke-WebRequest -Uri "http://<device-ip>" -TimeoutSec 5 -UseBasicParsing
    Write-Host "HTTP response: $($response.StatusCode)"
} catch {
    Write-Host "HTTP failed: $($_.Exception.Message)"
}
```

---

### Step 5 - Remote reboot (only if authorised and appropriate)

**Check the following BEFORE attempting any reboot:**
- Is this device in a maintenance window? (Check change calendar)
- Is there a "do not reboot without approval" note on the device in PRTG?
- Is this a production server where an unexpected reboot causes a service outage?

If out-of-band management is available (iDRAC, iLO, IPMI):
1. Log into the OOB management interface
2. Confirm the device is powered on but unresponsive at the OS level
3. Attempt a graceful OS restart first (if the OS is responsive via OOB console)
4. If the OS is completely unresponsive: hard reset is the last resort - document this as an emergency hard reset in the ticket (it is a Change action)

If no remote management is available:
- Contact on-site staff to physically check and power cycle the device
- Record the time you contacted them and their findings in the ticket

---

### Step 6 - Escalate if unresolved after 15 minutes

Escalate to L2 with this exact information:

```
PRTG ALERT ESCALATION
─────────────────────────────────────────
Device name    : [Name in PRTG]
Device IP      : [IP address]
Sensor(s) DOWN : [List all DOWN sensors]
Alert started  : [Time of first alert]
─────────────────────────────────────────
Scope check    : [Single device / Multiple devices (list them)]
Direct ping    : [Results — reply/timeout, packet loss %]
24h graph      : [What pattern you observed — sudden/gradual/flapping]
Reboot attempt : [Yes — result / No — reason]
─────────────────────────────────────────
Adjacent devices: [Status of devices in the same group/segment]
Business impact : [What service or users are affected right now]
```

---

## Procedure - HIGH LATENCY or PACKET LOSS Warning

When PRTG shows a WARNING (yellow) for latency or packet loss rather than full DOWN:

```cmd
:: Extended ping test - 100 packets gives a meaningful loss percentage
ping <device-ip> -n 100

:: Pathping - shows per-hop latency and packet loss across the route
:: Takes 2–5 minutes to run - worth the wait for WAN issues
pathping <device-ip>

:: Traceroute - shows each hop in the path and where latency is introduced
tracert <device-ip>
```

**Reading pathping output:**
- Loss at the final hop only: the target device has high load
- Loss starting at a specific intermediate hop: that hop (or the link after it) is degraded
- Loss increasing at every hop after a specific point: congestion at that point

Document the pathping output in the ticket - it shows exactly where the problem is.

---

## PRTG Threshold Reference - Standard Sensor Defaults

These thresholds are starting points. They should be tuned to a two-week baseline
for each device after initial deployment. Generic defaults generate false positives.

| Sensor | Warning Threshold | Down Threshold | Notes |
|--------|------------------|----------------|-------|
| Ping (latency) | > 50ms average | > 200ms or > 5% loss | Tune per device — servers on LAN should be < 5ms |
| CPU Load | > 80% for 5 min | > 95% for 5 min | Brief spikes are normal; sustained is not |
| Memory | > 85% utilised | > 95% utilised | Physical RAM only — page file is separate |
| Disk Free Space | < 20% free | < 10% free | Critical servers may need lower warning threshold |
| HTTP Response | > 5 seconds | Timeout / non-200 response | Tune to application baseline |
| SNMP Interface Traffic | > 80% utilisation | > 95% utilisation | Check for broadcast storms at high utilisation |
| Windows Service | Stopped | Not running after restart attempt | Critical services: Stopped = immediate P1 |


---