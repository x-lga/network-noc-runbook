# Procedure 06 - Escalation Criteria and Handoff Standard

**ITIL 4 Alignment:** Incident Management — escalation is a controlled, documented action, not an admission of failure.

---

## When to Escalate - Decision Tree

```
Is the issue resolved at L1?
    YES → Close ticket with resolution documentation
    NO  → Has the SLA window elapsed?
               YES → Escalate immediately
               NO  → Continue L1 steps

Multiple users affected?
    YES → Escalate immediately (potential P1)
    NO  → Continue single-user triage

Security indicator present? (phishing, malware, breach, data leak)
    YES → Escalate immediately AND notify security lead
    NO  → Continue standard triage

Root cause cannot be identified within 15 minutes at L1?
    YES → Escalate — do not hold a P2/P1 ticket at L1 trying to be a hero
    NO  → Document steps and continue
```

---

## L2 Escalation Handoff Template

When escalating a ticket, the handoff note must include ALL of the following:

```
TICKET: [ID]
PRIORITY: [P1/P2/P3/P4]
CALLER: [Name] | [Department] | [Contact]
ISSUE: [One-sentence description]

TIMELINE:
  - [Time] Issue reported by user
  - [Time] L1 triage began
  - [Time] [Step 1 taken and result]
  - [Time] [Step 2 taken and result]
  - [Time] [Step 3 taken and result]

CURRENT STATE:
  [What is happening right now]

REASON FOR ESCALATION:
  [Why this is not resolvable at L1]

URGENCY:
  [Business impact — what cannot the user/team do right now]
```

---

## Escalation Contacts

| Tier | Scope | Contact Method |
|------|-------|----------------|
| L2 - Systems | AD, server, on-site hardware | Ticket queue + Slack ping |
| L2 - Network | Routing, switches, WAN | Ticket queue + phone for P1 |
| L2 - Security | Any security incident | Direct phone call for P1, ticket for P2+ |
| L3 - Vendor | Hardware under warranty | Open vendor support case, attach serial |

**For P1: Always call L2. Do not rely on ticket queue alone.**


---