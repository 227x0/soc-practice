> Back to [case summary](README.md)

# Detection

> **Note on timestamps:** The Wazuh event export covers **2026-11-09 22:17:54 – 23:09:50**. Times below are from the export; the host's `/var/log/auth.log` is UTC, while the SIEM UI in this lab displays local time (UTC+1). The original lab notes used approximate times.

## What the SIEM saw

### Attack timeline

| # | Time (export) | Event | Source | Detection |
|---|---|---|---|---|
| 1 | 22:37:03.004 / 22:37:03.008 | HTTP fingerprinting (`curl`) and SQLi login bypass happen in the same second | Wazuh agent / Suricata | 1× rule **31164** — `SQL injection attempt` (L6); 1× rule **86601** — `ET HUNTING curl User-Agent to Dotted Quad` |
| 2 | 22:37:11.011 – 22:37:17.081 | nmap scan / port probes from `192.168.100.20` | Suricata | 25× rule **86601** (ET SCAN / ET HUNTING user-agents and port probes across the whole window) |
| 3 | 23:00:08.331 | nginx error message | nginx error log | 1× rule **31301** |
| 4 | 23:09:50.657 | Web server 500 Internal Error | Wazuh agent | 1× rule **31122** (L5) |
| 5 | shortly after 22:37 | `GET /api/Users` with stolen Bearer token | **no alert in SIEM** | ⚠️ visibility gap — see below |

### Unrelated host noise (same export window)

These events are from normal host maintenance / agent lifecycle and are **not part of the attack**:

| Time (export) | Event | Rule IDs |
|---|---|---|
| 22:17:54.000 | Wazuh agent stopped | 506 |
| 22:35:22.859 | Wazuh agent started | 503 |
| 22:35:28.051 – 22:37:13.010 | Interactive PAM sessions + sudo | 5501, 5502, 5402, 5403 |
| 22:35:29.097 – 22:35:29.177 | System audit events | 516 |
| 22:40:11.590 / 22:40:13.578 | USB disconnect / attach | 81102, 81101 |

They are kept in the export for completeness, but L1 triage should not mix them into the incident timeline.

## Event breakdown (attack-related events only)

| Count | Rule ID | Level | Description |
|---|---|---|---|
| 25 | 86601 | 3 | Suricata: Alert — ET SCAN / ET HUNTING user-agents and port probes |
| 1 | 31164 | 6 | SQL injection attempt |
| 1 | 31122 | 5 | Web server 500 error code (Internal Error) |
| 1 | 31301 | 3 | Nginx error message |

**Total attack-related alerts:** 28 events. The remaining 21 events in the export are host/agent noise listed above.

## Key alert example (Suricata → Wazuh)

Actual Wazuh-decoded alert from the export, rule **86601**:

```json
{
  "rule": {
    "id": "86601",
    "level": 3,
    "description": "Suricata: Alert - ET SCAN Suspicious inbound to MSSQL port 1433"
  },
  "data": {
    "src_ip": "192.168.100.20",
    "dest_ip": "192.168.100.24",
    "dest_port": "1433",
    "alert.signature": "ET SCAN Suspicious inbound to MSSQL port 1433",
    "alert.signature_id": "2010935",
    "alert.severity": "2",
    "flow.pkts_toserver": "1",
    "flow.pkts_toclient": "0"
  }
}
```

**Analyst reading:** one packet to server, none to client, Suricata severity Informational → this is a *port probe during a scan*, **not** evidence that MSSQL is open. Correlated with scan results, port 1433 answered RST. Alert ≠ fact.

## Evidence

- [`iocs/wazuh-events.csv`](iocs/wazuh-events.csv) — raw Wazuh export used for counts and timestamps.
  > **Limitation:** this export does not include `data.srcip` or `full_log`. A better re-export from the SIEM would include those fields for full transparency.
- [`iocs/wazuh-threat-hunting-report.pdf`](iocs/wazuh-threat-hunting-report.pdf) — 5-page Wazuh Threat Hunting report for agent `001`.
- [`iocs/screenshots/wazuh-agent-overview.png`](iocs/screenshots/wazuh-agent-overview.png) — agent dashboard screenshot (events evolution + MITRE tactics).

![Wazuh agent overview — events evolution and MITRE tactics](iocs/screenshots/wazuh-agent-overview.png)

### Raw export sample

First and last lines of the Wazuh event export used for this case:

```csv
timestamp,agent.name,rule.description,rule.level,rule.id
"Nov 9, 2026 @ 22:17:54.000",u3,Wazuh agent stopped.,3,506
...
"Nov 9, 2026 @ 23:09:50.679",u3,sshd: authentication failed.,5,86601
```

Full file: [`iocs/wazuh-events.csv`](iocs/wazuh-events.csv).

## Visibility gap: `GET /api/Users`

The final exfiltration request was sent with a valid admin JWT. In the current pipeline:

- The request hit nginx and was logged to `access.log`.
- No Wazuh rule fires on "admin API call from a new source IP", so there is **no alert**.
- Whether the raw `access.log` line reached Wazuh archives depends on `logall` / file shipping; this has not been verified on the live SIEM yet.

**Verdict for the case:** even if the log line is stored, there is no detection rule for it — so from an L1 analyst's point of view the stage is **not visible as an alert**. The gap should be documented and handed to detection engineering.

## MITRE ATT&CK Mapping

- **T1595.002 Active Scanning: Vulnerability Scanning** — nmap SYN scan
- **T1046 Network Service Discovery** — port/service enumeration
- **T1083 File and Directory Discovery** — `/ftp` enumeration via robots.txt
- **T1190 Exploit Public-Facing Application** — path traversal + SQLi (Juice Shop)
- **T1078 Valid Accounts** — use of the forged admin JWT
- **T1005 Data from Local System** — `/api/Users` dump (PII exfiltration)
