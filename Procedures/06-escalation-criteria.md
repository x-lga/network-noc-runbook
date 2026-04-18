# Procedure 06 - Escalation Criteria and L2 Handoff Standard

**ITIL 4 Alignment:** Incident Management - controlled, documented escalation  
**Cert alignment:** ITIL 4 Foundation, CompTIA Network+  
**Last reviewed:** 2026-07

---

## Philosophy

Escalation is not failure. In ITIL 4 Incident Management, escalation is the correct
process response when an incident exceeds L1 resolution capability. Holding a P1 or P2
at L1 because you want to solve it yourself is the real failure - it breaches SLAs,
delays resolution, and wastes the L2 engineer's time when they eventually receive a
half-investigated ticket.

The goal is to escalate at the right moment with complete documentation, so L2 spends
their time solving the problem - not re-gathering information that L1 already had.

---

## When to Escalate - Decision Criteria

Escalate to L2 **immediately** when any of the following is true:

**1. SLA breach approaching or reached**
The ticket has been open for a duration approaching the resolution SLA for its priority
and the issue is not resolved. Do not wait for the SLA to expire — escalate with buffer.

**2. Root cause unidentifiable at L1**
L1 cannot determine the cause of the issue within the appropriate triage window:
- P1/P2: 15 minutes of active L1 investigation
- P3/P4: 30 minutes of active L1 investigation

**3. Multiple users or sites affected**
Any single incident affecting more than one user simultaneously should be considered
potential P1 infrastructure failure until proven otherwise.

**4. Security indicator of any kind**
- Phishing email confirmed or suspected (especially if credentials were entered)
- Malware detected - active execution suspected
- Unusual account activity or failed auth spikes
- Data accessed without authorisation
- Unusual outbound network traffic

**5. Account re-enablement request**
A disabled (not locked) account requires re-enabling. This requires manager confirmation
and L2 verification - it cannot be done at L1.

**6. Physical hardware failure**
Any issue requiring physical repair, component replacement, or on-site server work.

**7. Infrastructure change required**
Resolving the issue requires modifying: firewall rules, GPO settings, DNS records,
DHCP scopes, VLAN configurations, Azure policies, or any other infrastructure component.

**8. Azure platform-initiated issue**
Azure Resource Health shows a platform-initiated unavailability. This is Microsoft's
infrastructure issue - L1 cannot resolve it, and creating tickets with Azure support
requires L2/L3 involvement.

---

## Escalation Urgency Guide by Priority

| Priority | Escalation Method | Timing |
|----------|------------------|--------|
| P1 - Critical | Phone call to L2 immediately + create ticket simultaneously | Within 5 minutes of classification |
| P2 - High | Update ticket with escalation note + Slack/Teams ping to L2 | Before SLA response window expires |
| P3 - Medium | Update ticket with escalation note + assign to L2 queue | Before SLA response window expires |
| P4 - Low | Update ticket with escalation note | Normal queue assignment |

**For P1: Always call. Do not rely on the ticket system - the ticket may not be seen immediately.**

---

## L2 Escalation Handoff Template

Copy this template into the ticket notes every time you escalate.
Do not escalate without filling every field.
An incomplete handoff is the primary cause of wasted L2 diagnostic time.

```
════════════════════════════════════════════════════
  L2 ESCALATION HANDOFF
════════════════════════════════════════════════════
TICKET ID     : [INC-YYYYMMDD-XXXX]
PRIORITY      : [P1 / P2 / P3 / P4]
ESCALATED BY  : [Your name and role]
ESCALATED AT  : [HH:MM DD/MM/YYYY]
────────────────────────────────────────────────────
CALLER        : [Full name]
DEPARTMENT    : [Department]
CONTACT       : [Phone extension or direct line]
ASSET         : [Computer name / server / system]
LOCATION      : [Physical or network location]
────────────────────────────────────────────────────
ISSUE         : [One clear sentence - symptom as reported by the caller]
────────────────────────────────────────────────────
INVESTIGATION TIMELINE:
  [HH:MM] Ticket logged
  [HH:MM] L1 assigned and triage started
  [HH:MM] [First step taken] - [Result]
  [HH:MM] [Second step taken] - [Result]
  [HH:MM] [Third step taken] - [Result]
  [HH:MM] Escalation decision made
────────────────────────────────────────────────────
CURRENT STATE :
  [Describe exactly what is happening right now -
  not the original symptom, the current state after L1 steps]
────────────────────────────────────────────────────
REASON FOR ESCALATION:
  [Specific reason why this cannot be resolved at L1 -
  not just "unresolved" - WHY it cannot be resolved at L1]
────────────────────────────────────────────────────
BUSINESS IMPACT:
  [What the user or team cannot do right now -
  what service is unavailable, what deadline is affected]
════════════════════════════════════════════════════
```

---

## Ticket Closure Standard

Before closing any ticket at L1 (or confirming L2 resolution):

- [ ] User confirmed issue is resolved - do not close without confirmation
- [ ] Every step taken is documented in the ticket with timestamps
- [ ] Resolution is documented - not "fixed," but exactly what was done
- [ ] If escalated: L2 resolution notes are in the ticket
- [ ] If a recurring issue was identified: Problem ticket has been raised and linked
- [ ] If a change was made: Change ticket reference is noted in the Incident ticket
- [ ] If security-related: Security team acknowledgement is in the ticket


---