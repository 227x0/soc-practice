> All IPs and accounts are fictional and used only in a home lab.

# IOCs

This folder contains artifacts from the case.

## Files

- `auth.log.fragment` — reconstructed fragment of `/var/log/auth.log` from the target host `u3`.
- `wazuh-events.csv` — raw Wazuh event export used for the timeline and counts.
- `wazuh-threat-hunting-report.pdf` — official 5-page Wazuh Threat Hunting report for agent `u3` (PDF export).
- `screenshots/` — Wazuh SIEM screenshots captured during the lab.

## Notes

- The `auth.log.fragment` is reconstructed from the Wazuh alert timeline (`~37 alert events` followed by a successful PAM session).
- Screenshots show the Wazuh dashboard summary, rule 40112 alert details, and MITRE ATT&CK mapping.
