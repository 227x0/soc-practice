# Attacker Upgrade (Kali .20)

Installed to `~/tools`, symlinked into `/usr/local/bin`. All versions verified after install.

| Tool | Version | Purpose | First command |
|---|---|---|---|
| nuclei | 3.11.1 | Vulnerability scanner, 8k+ YAML templates | `nuclei -u http://192.168.100.24` |
| httpx | 1.12.0 | Fast HTTP prober: status, title, tech-detect | `echo 192.168.100.24 \| httpx -title -tech-detect` |
| subfinder | 2.17.0 | Passive subdomain discovery (OSINT sources) | `subfinder -d <your-domain>` |
| katana | 1.8.0 | Web crawler — endpoint discovery | `katana -u http://192.168.100.24` |
| naabu | 2.7.0 | Fast port scanner (complements nmap) | `naabu -host 192.168.100.24 -top-ports 1000` |
| feroxbuster | 2.13.1 | Directory/file fuzzing | `feroxbuster -u http://192.168.100.24 -w /usr/share/seclists/Discovery/Web-Content/common.txt` |
| sherlock | 0.16.2 | Username enumeration across social platforms | `sherlock <username>` |
| SecLists | (apt `seclists`) | Wordlists for everything above | `/usr/share/seclists/` |

## Verification run

Generates a small, recognizable burst of web requests in the SIEM — a free correlation exercise for L1.

```
$ httpx -u http://192.168.100.24 -title -tech-detect -status-code
http://192.168.100.24 [200] [OWASP Juice Shop] [Nginx:1.28.3,Ubuntu]

$ echo http://192.168.100.24 | katana -silent -d 2 -timeout 5
http://192.168.100.24
http://192.168.100.24/main.js
http://192.168.100.24/polyfills.js
http://192.168.100.24/scripts.js
...
```

## Honest limitation

katana's static crawl of the Juice Shop (an Angular SPA) finds JS bundles but **not** `/ftp` or `/api` — those sit behind client-side rendering and XHR calls. Finding them needs `katana -js-crawl` (headless) or authenticated crawling.

**Lesson for L1:** tool output ≠ full attack surface. The manual enumeration that found `/ftp` in SOC-2026-002 is still a required skill; automation amplifies an analyst, it does not replace one.
