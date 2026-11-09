# SOC Practice Cases

> Home cybersecurity lab: hands-on SOC L1/L2 cases.  
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

## Disclaimer

All IPs, accounts, and passwords are fictional and used only in an isolated home lab. Do not use the described techniques against systems you do not own or without written permission.
