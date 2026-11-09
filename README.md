# SOC Practice Cases

> Home cybersecurity lab: hands-on SOC L1 cases.  
> Each case covers the full cycle: **attack → detection → triage → escalation → response**.

## Cases

| # | Name | Level | SIEM / Tools | Status |
|---|------|-------|--------------|--------|
| 01 | [SSH Brute-Force: from attack to L1 escalation](./cases/01-ssh-bruteforce) | Junior / L1 | Wazuh, VirtualBox, Kali, Ubuntu | ✅ Ready |

## Purpose of this repository

- Practice alert analysis and incident investigation.
- Collect escalation templates and response playbooks.
- Document lab mistakes and infrastructure lessons learned.

## How to read a case

Each folder contains:

- `README.md` — full case write-up.
- `iocs/` — IOCs, logs, screenshots (if any).
- `templates/` — ticket templates and checklists.

## How to add a new case

1. Copy `cases/00-TEMPLATE/` to a new folder, e.g. `cases/02-phishing/`.
2. Fill in `README.md` following the template sections.
3. Add evidence to `iocs/`:
   - `iocs/screenshots/` — SIEM screenshots, terminal captures;
   - `iocs/wazuh-events.csv` — raw event export (optional);
   - `iocs/auth.log.fragment` — relevant log fragment (optional);
   - any PDF reports or other artifacts.
4. Update this table in `README.md` with the new case.
5. Commit and push:
   ```bash
   git add cases/02-phishing/
   git commit -m "Add case 02: phishing"
   git push origin master
   ```

## Disclaimer

All IPs, accounts, and passwords are fictional and used only in an isolated home lab. Do not use the described techniques against systems you do not own or without written permission.
