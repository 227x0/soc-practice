> Replace this block with any important notes (e.g., "Home lab only", "No production systems harmed").

# SOC Practice #N — [Case Title]

> One-sentence summary of the case. What happened and what was detected?

## Metadata

| Field | Value |
|---|---|
| **Case ID** | SOC-YYYY-NNN |
| **Date** | YYYY-MM-DD |
| **Difficulty** | Junior / L1 |
| **Domain** | SOC, Incident Response, ... |
| **MITRE ATT&CK** | TXXXX Technique Name |
| **SIEM** | Wazuh X.YZ |

## Summary

Describe the incident in 3–5 sentences. Include attacker action, target, detection, and final verdict.

---

## Lab Environment

| Role | VM | IP | OS / Software |
|---|---|---|---|
| Attacker | `...` | `x.x.x.x` | ... |
| Target | `...` | `x.x.x.x` | ... |
| SIEM | `...` | `x.x.x.x` | ... |

Network / topology notes go here.

---

## Attack Scenario

Explain the attack step by step. Include commands if relevant:

```bash
# attacker
command --example
```

### Why this works

- Vulnerability #1
- Vulnerability #2

---

## Detection in [SIEM]

Describe what the SIEM saw. Include alert counts and a timeline.

| Time (UTC) | Rule ID | Level | Description |
|---|---|---|---|
| ... | ... | ... | ... |

### Key alert

Which alert is the smoking gun? Why?

### MITRE ATT&CK mapping

- **TXXXX** Technique Name (Tactic)

### Evidence

- `iocs/screenshots/` — SIEM screenshots
- `iocs/wazuh-events.csv` — raw event export (if applicable)
- `iocs/auth.log.fragment` — log fragment (if applicable)
- `iocs/wazuh-threat-hunting-report.pdf` — PDF report (if applicable)

---

## L1 Analyst Triage

1. **Validation.** Why is this not a false positive?
2. **False-positive check.** What did you rule out?
3. **Enrichment.** What context did you add?
4. **Verdict.** True positive / false positive, severity.

### L1 checklist

- [ ] Confirm the malicious pattern.
- [ ] Check source/target IP reputation and ownership.
- [ ] Check host/user alert history.
- [ ] Set severity.
- [ ] Prepare a ticket for L2.

---

## Escalation to L2 (Ticket Template)

```
[ESCALATION][P?] [Short title]
Rule: [Rule ID] "[Description]" (L[Level])
Time: [UTC timestamp]
Target: [host / account]
Source: [IP / host]
Evidence: [what proves the incident]
Context: [enrichment]
Requested: [actions for L2]
```

---

## Response (L2)

### Immediate containment

```bash
# commands or playbooks
```

### Eradication & investigation

- Step 1
- Step 2

### Automation hardening

- What will prevent recurrence?

---

## IOCs (lab)

| Type | Value |
|---|---|
| Source IP | `x.x.x.x` |
| Target | `x.x.x.x:port` |
| Account | `...` |
| Tool | `...` |
| Rule | `...` |

---

## Takeaways

- Lesson #1
- Lesson #2

---

## References

- [Link 1](https://example.com)
