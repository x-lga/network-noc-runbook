# Procedure 01 — Connectivity Troubleshooting

**Category:** Network Incident  
**ITIL 4 Priority:** P2 (single user) / P1 (multiple users or site-wide)  

---

## Symptoms

- User reports "no internet" or "cannot reach network resources"
- Browser shows "ERR_NAME_NOT_RESOLVED" or "This site can't be reached"
- Mapped drives disconnected

---

## Layered Troubleshooting Procedure (OSI Model Approach)

Work from Layer 1 upward. Do not skip layers.

### Layer 1 — Physical

- [ ] Check NIC lights on the back of the PC: link light (solid) + activity light (blinking)
- [ ] Reseat ethernet cable at both ends
- [ ] Try a different ethernet cable
- [ ] If wireless: check if Wi-Fi is enabled (Windows: taskbar Wi-Fi icon)
- [ ] If wireless: ensure the user is connected to the correct SSID (not a neighbour's network)

**If Layer 1 passes:** Proceed to Layer 2

---