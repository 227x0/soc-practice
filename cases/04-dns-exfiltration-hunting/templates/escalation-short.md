> Back to [case summary](../README.md) · [full L2 report](full-report.md)

# Escalation to L2 (short ticket)

```
[ESCALATION][P2] DNS exfiltration from internal host 192.168.100.20
Case: SOC-2026-004 / lab incident INC-2026-0003
Time: 2026-11-18 23:04:35Z (00:04:35 local CET)
Source: 192.168.100.20 -> 192.168.100.24:53 (UDP/53)
Pattern: 13 DNS A-queries in <1s, chunks numbered 0-12,
         subdomains are base64-like (50-char), domain exfil.lab
         (unknown to the environment, REFUSED by the sink)
Alerts fired: NONE (no matching rule; archives only)
Evidence: iocs/wazuhe-arhive-18.11.2026(00.04.35).json (39 events),
          iocs/discover-screenshot.png
Assessment: data is ENCODED (base64), not encrypted - content
            recoverable by reassembly (strip 0-12 prefixes, join, decode)
Requested: confirm scope (24h lookback on .20), decode payload,
           check host .20 for the sender process, egress-filtering proposal
```
