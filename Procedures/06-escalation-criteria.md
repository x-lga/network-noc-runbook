# Procedure 06 — Escalation Criteria and Handoff Standard

**ITIL 4 Alignment:** Incident Management — escalation is a controlled, documented action, not an admission of failure.

---

## When to Escalate — Decision Tree

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