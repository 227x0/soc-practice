> All IOCs here are fictional / lab-only unless otherwise stated.

# IOCs

## Artifacts

| File | Description |
|---|---|
| `wazuh-events.csv` | Raw 49-event Wazuh export for agent `001` |
| `wazuh-threat-hunting-report.pdf` | 5-page Wazuh Threat Hunting report |
| `screenshots/wazuh-agent-overview.png` | Agent dashboard: events evolution + MITRE tactics |

## Indicators

| Type | Value |
|---|---|
| Source IP | `192.168.100.20` (Kali) |
| Target | `192.168.100.24:80` (nginx → Juice Shop) |
| Scan signature | `ET SCAN Suspicious inbound to MSSQL port 1433`, sid 2010935 |
| Wazuh decoder rule | 86601 (Suricata alert), level 3 |
| SQLi rule | 31164 (SQL injection attempt), level 6 |
| Web error rule | 31122 (Web server 500 error), level 5 |
| SQLi payload | `' OR 1=1--` in JSON body of POST `/rest/user/login` |
| Artifact | Admin JWT for `admin@juice-sh.op` (role: admin), obtained via SQL injection |
| Exfil endpoint | `GET /api/Users` (200, full user list) |
| Leaked doc | `/ftp/acquisitions.md` (200) |
| Behavior pattern | scan → robots.txt → `/ftp` → SQLi login → admin API burst, all within ~40 min from one src IP |
