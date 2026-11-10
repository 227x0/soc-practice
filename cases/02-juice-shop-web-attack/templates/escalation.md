> Back to [case summary](../README.md) · [full L2 report](full-report.md)

# L2 Escalation Ticket (short)

```
[ESCALATION][P1/P2] Web app compromise: SQLi auth bypass + PII exfiltration
Case ID: SOC-2026-002
Escalated by: SOC L1
Date: 2026-11-09

Verdict: True positive. Confirmed multi-stage web attack on OWASP Juice Shop.

Scope:
- Target: u3 (192.168.100.24) — nginx/Juice Shop, agent 001
- Source: 192.168.100.20 (internal Kali — verify CMDB before blocking)
- Affected: admin account, full user DB, /ftp/acquisitions.md

Timeline (UTC):
- 22:37:03 — HTTP recon + SQLi login bypass (rule 31164, L6)
- 22:37:11–23:09:50 — nmap/curl scan activity (25× Suricata rule 86601)
- shortly after 22:37 — GET /api/Users with stolen admin JWT → no SIEM alert

Impact:
1. Confidential doc leaked: /ftp/acquisitions.md
2. Account takeover: admin JWT issued via SQLi
3. PII exfiltration: full user DB via admin API

Visibility gap: no rule for admin-JWT API abuse; log coverage not verified.

Requested actions:
1. Block 192.168.100.20 after CMDB check.
2. Invalidate JWTs/sessions; force password reset for admin@juice-sh.op.
3. Scope review of /api/* and /ftp/* in nginx access.log.
4. Deploy WAF + app-level admin API audit logging.
```

Full evidence package and root-cause notes: [`full-report.md`](full-report.md).
