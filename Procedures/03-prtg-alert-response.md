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

---

### Step 2 — Check if adjacent devices are affected

- In PRTG device tree: expand the parent device group
- If multiple devices on the same switch are DOWN → switch or uplink issue (not individual device)
- If only one device is DOWN → isolated device failure

---

### Step 3 — Check PRTG history

- Click the sensor > Graph > Last 24 hours
- Look for: did this drop suddenly or degrade gradually?
- Sudden drop = likely power or link failure
- Gradual degradation = resource exhaustion or hardware wear

---

### Step 4 — Attempt remote reboot

If you have access to the device's management interface (iDRAC, iLO, IPMI):

```
1. Log into OOB management interface
2. Confirm device shows as powered on but unresponsive
3. Issue a graceful restart if possible
4. If graceful restart unavailable: hard reset (document this in the ticket)
```

If no management interface:
- Contact on-site staff to physically check/reboot the device
- Log the contact attempt in the ticket

---
