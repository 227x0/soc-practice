> Back to [case summary](../README.md)

# L2 Escalation Ticket (short)

```
[ESCALATION][P2] DNS tunneling: encoded data exfiltration over DNS
Case ID: SOC-2026-004
Escalated by: SOC L1
Date: 2026-11-18

Verdict: True positive. DNS tunneling confirmed in wazuh-archives-*,
no alert rule fired.

Scope:
- Affected host:  192.168.100.20 (internal, Kali — data source, not in CMDB)
- Destination:    192.168.100.24:53 (sink, receiving resolver)
- Affected asset: data on host .20 — ~500 bytes encoded into 13 chunks;
                  exact content to be confirmed by decoding (see actions)

Timeline (UTC):
- 23:04:35Z — burst of 13 A-queries, chunks 0-12, to exfil.lab (0 alerts)

Impact:
- Data confidentiality breached (small volume, single host)
- Detection gap: whole class of attack invisible to alerting

Requested actions:
1. Isolate host 192.168.100.20 (EDR network isolation) — domain blocking
   alone is bypassable by renaming the domain.
2. Decode the 13 chunks (strip N. prefixes, join, base64 -d) to establish
   what data was taken.
3. 24h lookback on host .20 + other hosts for the same pattern.
4. Write/verify SIEM rules for DNS tunneling patterns (burst to one domain,
   long labels, REFUSED/NXDOMAIN ratio, entropy).
```

Full evidence package: [`../iocs/`](../iocs).
