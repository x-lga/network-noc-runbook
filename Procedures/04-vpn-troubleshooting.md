# Procedure 04 — VPN Troubleshooting

**Category:** Remote Access  
**ITIL 4 Priority:** P2 (single user) / P1 (multiple users)

---

## Symptoms

- User cannot connect to VPN
- VPN connects but disconnects frequently
- VPN connected but cannot reach internal resources

---

## Step 1 — Scope Check (Critical First Step)

```
Q: Is this one user or multiple users?

Multiple users → P1 escalation immediately. Server-side issue.
    - Notify L2/L3 immediately
    - Check VPN server status in monitoring dashboard
    - Do not spend time on client-side steps

One user → continue with client-side procedure below
```

---

## Step 2 — Local Connectivity Check

```cmd
# Confirm local internet works before blaming VPN
ping 8.8.8.8

# If ping fails: local internet issue — see Procedure 01 first
```

---