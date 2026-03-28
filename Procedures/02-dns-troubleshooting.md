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