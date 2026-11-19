> Back to [case summary](../README.md) · [short escalation ticket](escalation-short.md)

# Full L2 Report

```
[ESCALATION][P2] DNS exfiltration: encoded file transfer over DNS
Case ID: SOC-2026-004
Escalated by: SOC L1
Date: 2026-11-18 (attack) / 2026-11-19 (report)

## Verdict
True positive, P2. Confirmed exfiltration of a ~500-byte file from an internal
host, encoded into DNS queries. Single source, small volume, no spread.

## Scope
| | |
|---|---|
| Source host | 192.168.100.20 (internal; no CMDB ownership found) |
| Sink | 192.168.100.24:53 (dnsmasq query-log receiver in lab; in prod = attacker's NS via victim's resolvers) |
| Channel | DNS A-queries, UDP/53 |
| Data | ~500 bytes, base64-encoded, split into 13 numbered chunks of 50 chars |

## Timeline (UTC)
| Time | Event |
|---|---|
| 23:04:35.139Z | first chunk (0) sent |
| 23:04:35.139-.201Z | chunks 0-12 sent + REFUSED responses (<1s total) |
| ~23:04:52Z | events indexed (ingestion delay ~9-17s) |
| manual hunt | found in wazuh-archives-* (0 alerts fired) |

## Impact
1. Data confidentiality: ~500 bytes of (test) data left the host.
2. Detection gap: the entire class of attack was invisible (no rule + archives
   were off until SOC-2026-003).
3. Process: no 24h lookback performed at L1 - flagged as enrichment gap.

## Evidence
- iocs/wazuhe-arhive-18.11.2026(00.04.35).json - 39 events (Suricata req/resp, dnsmasq)
- iocs/dig-output.txt - attacker-side capture (chunks 0,6,12)
- iocs/discover-screenshot.png - hunt view
- Runbook: soc-lab/l1-ops/runbooks/rb-dns-anomaly.md

## Payload recovery (for L2)
strip leading "N." from each subdomain, order by N=0..12, join the 50-char
chunks, replace 'X'->'+','0'->'/' (the tr sanitization), base64 -d.
Expected: the original secret.txt content.

## Requested actions
1. 24h lookback on 192.168.100.20 (other bursts / suspicious domains).
2. Host check on .20: which process sent the queries (Velociraptor hunt).
3. Egress filtering: force DNS via internal resolver only; sink unknown TLDs.
4. Detection: archive rule set for DNS-burst/entropy pattern (severity >= 8).

## Root causes
- No detection rule for DNS tunneling patterns.
- Archives disabled (fixed in SOC-2026-003).
- No egress restriction on 53/udp (lab).

---
*Prepared by L1 per case SOC-2026-004. Lab simulation; the receiving side was
local (u3). In production the sink is the attacker's authoritative NS, reached
through the victim's own resolvers.*
```
