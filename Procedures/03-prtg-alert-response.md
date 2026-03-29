# Procedure 03 — PRTG Alert Response

**Category:** NOC Event Management  
**ITIL 4 Priority:** Determined by sensor type and alert threshold 

---

## Alert States in PRTG

| Status | Colour | Meaning |
|--------|--------|---------|
| Up | Green | Device/service operating within thresholds |
| Warning | Yellow | Approaching threshold — monitor closely |
| Down | Red | Device/service unreachable or threshold breached |
| Paused | Blue | Manually paused — check if intentional |
| Unknown | Grey | No data received — sensor misconfigured or agent issue |

---

## Procedure — Sensor Shows DOWN

### Step 1 — Verify the alert is real

Before acting, confirm this is not a sensor misconfiguration or transient blip.

```cmd
# Ping the device directly from your NOC workstation
ping <device-ip> -n 10

# If ping responds: sensor may be misconfigured (e.g., wrong SNMP community string)
# If ping fails: device is genuinely unreachable
```