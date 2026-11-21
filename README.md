# SOC Practice Cases

![Cases](https://img.shields.io/badge/cases-4-e94560)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh%204.14-blue)
![IDS](https://img.shields.io/badge/IDS-Suricata-red)
![EDR](https://img.shields.io/badge/EDR-Velociraptor-green)
![MITRE](https://img.shields.io/badge/MITRE%20ATT%26CK-mapped-0f3460)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

> Home cybersecurity lab: hands-on SOC L1 cases.  
> Each case covers the full cycle: **attack → detection → triage → escalation → response**.

## Cases

| # | Name | Level | SIEM / Tools | Status |
|---|------|-------|--------------|--------|
| 01 | [SSH Brute-Force: from attack to L1 escalation](./cases/01-ssh-bruteforce) | Junior / L1 | Wazuh, VirtualBox, Kali, Ubuntu | ✅ Ready |
| 02 | [Juice Shop Web Attack: recon → path traversal → SQLi → data exfiltration](./cases/02-juice-shop-web-attack) | Junior / L1 | Wazuh, Suricata, VirtualBox, Kali, Ubuntu, OWASP Juice Shop | ✅ Ready |
| 03 | [Lab Upgrade: Arsenal & Telemetry](./cases/03-lab-upgrade-arsenal-telemetry) | Junior / L1 | Wazuh, auditd, ProjectDiscovery stack, VirtualBox | ✅ Ready |
| 04 | [DNS Exfiltration: From Attack to L1 Escalation](./cases/04-dns-exfiltration-hunting) | Junior / L1 | Wazuh, Suricata, dnsmasq, bash | ✅ Ready |
| 05 | [Post-Exploitation Hunt: SSH Compromise & Persistence](./cases/05-post-exploitation-hunt) | Junior / L1 (L2 elements) | Wazuh, auditd lab_exec, Velociraptor, ausearch | ✅ Ready |

## Purpose of this repository

- Practice alert analysis and incident investigation.
- Collect escalation templates and response playbooks.
- Document lab mistakes and infrastructure lessons learned.

## Disclaimer

All IPs, accounts, and passwords are fictional and used only in an isolated home lab. Do not use the described techniques against systems you do not own or without written permission.
