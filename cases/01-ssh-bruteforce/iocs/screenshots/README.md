# Screenshots

Real evidence screenshots from the lab Wazuh SIEM.

## Files

- `wazuh-dashboard.png` — dashboard summary: 40 total alerts, 37 authentication failures, 1 authentication success, 1 level-12+ alert.
- `wazuh-rule-40112-details.png` — detailed view of correlation rule 40112 (`Multiple authentication failures followed by a success`), showing source IP `192.168.100.20`, target `u3`, and `Accepted password` log entry.
- `wazuh-rule-40112-mitre.png` — MITRE ATT&CK mapping for rule 40112: **T1110 Brute Force** and **T1078 Valid Accounts**, level 12.

## Redaction check

Before uploading, I verified that no real personal email, public IP, or unrelated hostname is visible. Only lab artifacts are shown (`192.168.100.20`, `192.168.100.24`, account `u3`).
