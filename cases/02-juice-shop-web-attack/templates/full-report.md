> Back to [case summary](../README.md) · [short escalation ticket](escalation.md)

# Full L2 Report

```
# [ESCALATION][P1] Web app compromise: SQLi auth bypass + PII exfiltration

**Case ID:** SOC-2026-002
**Escalated by:** SOC L1
**Date:** 2026-11-09

## Verdict

**True positive.** Confirmed multi-stage web attack on the Juice Shop application, ending in admin account takeover and mass PII exfiltration. Source IP has no legitimate role; activity pattern is a textbook kill-chain (recon → exploitation → exfiltration).

## Scope

| | |
|---|---|
| **Target host** | u3 (192.168.100.24) — Ubuntu, nginx/1.28.3, OWASP Juice Shop, Wazuh agent `001` |
| **Source IP** | 192.168.100.20 (internal — verify ownership in CMDB before blocking) |
| **Affected assets** | Juice Shop web app: admin account, full user database, `/ftp` documents |

## Timeline (UTC)

| Time | Event | Evidence |
|---|---|---|
| 22:37:03.008Z – 23:09:50Z | Scan / recon activity (nmap/curl) | Suricata rule 86601 (25 alerts); includes `ET SCAN Suspicious inbound to MSSQL port 1433` sid 2010935 |
| 22:37:03.004Z | POST `/rest/user/login`, body contains `' OR 1=1--` → 200 + admin JWT | Wazuh rule 31164 (SQL injection attempt, L6) + nginx access.log; recon and exploitation occurred within the same second |
| ~22:37Z | GET `/ftp/` → 200, GET `/ftp/acquisitions.md` → 200 | nginx access.log — confidential document exfiltrated |
| 23:00:08Z | nginx error message | Wazuh rule 31301 |
| 23:09:50Z | Web server 500 Internal Error | Wazuh rule 31122 (L5) |
| +minutes | GET `/api/Users` with `Authorization: Bearer <admin JWT obtained via SQLi>` → 200 | ⚠️ Not visible in SIEM — see visibility gap below — full user DB dump |

## Impact

1. **Data leak** — confidential business document (`acquisitions.md`) downloaded.
2. **Account takeover** — valid admin JWT issued for `admin@juice-sh.op` via SQL injection.
3. **PII exfiltration** — entire user table retrieved through the authenticated admin API.

## Evidence attached

- Wazuh event export for agent `001`: `iocs/wazuh-events.csv`
- 5-page Wazuh Threat Hunting report: `iocs/wazuh-threat-hunting-report.pdf`
- Agent overview screenshot: `iocs/screenshots/wazuh-agent-overview.png`
- Suricata alert example: `ET SCAN Suspicious inbound to MSSQL port 1433`, sid 2010935, severity 2, 1 pkt to server / 0 to client → probe only, port 1433 was closed
- nginx access.log lines for src `192.168.100.20` (GET `/`, `/robots.txt`, `/ftp/`, `/ftp/acquisitions.md`, POST `/rest/user/login`)

## ⚠️ Visibility gap (action needed)

`GET /api/Users` with an admin Bearer token produced **no SIEM alert**. Network IDS does not flag authenticated API traffic, and there is currently no Wazuh rule for "admin API call from a new source IP". The underlying nginx `access.log` line may have been shipped, but without a rule it is not visible to L1.

**Requested from engineering:** verify that nginx `access_log` for all vhosts is shipped by the Wazuh agent without rotation gaps; if `logall` is enabled, search archives for `/api/Users`; create an application-level audit rule for admin API access (who/what/when/src IP).

## Requested actions (L2 / IR)

1. **Containment:** block `192.168.100.20` at the firewall (after CMDB check).
2. **Eradication:** invalidate all active JWTs/sessions; force password reset, starting with `admin@juice-sh.op`; rotate app secrets.
3. **Scope review:** from nginx access.log — all `/api/*` and `/ftp/*` requests from the source IP; check `/ftp` for other exposed files.
4. **Detection improvements:** deploy WAF (ModSecurity + OWASP CRS); enable/verify Suricata SQLi ruleset; alert on behavior pattern (new src IP → admin-scope API burst within minutes of login).

## Root causes to fix (not L1 scope, flagging for awareness)

- SQL injection in `/rest/user/login` — app must use parameterized queries.
- Directory listing enabled on `/ftp` + sensitive files inside web root.
- Reliance on `robots.txt` as access control.
- Server version disclosure (`Server: nginx/1.28.3`).

---
*Prepared by L1 per case SOC-2026-002. No changes were made to any host.*
```
