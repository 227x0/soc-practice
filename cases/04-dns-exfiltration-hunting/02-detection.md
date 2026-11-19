> Back to [case summary](README.md)

# Detection

Evidence export: [`iocs/wazuhe-arhive-18.11.2026(00.04.35).json`](iocs/wazuhe-arhive-18.11.2026(00.04.35).json)
— event logs from `wazuh-archives-*` captured at the time of the incident.

## What the SIEM saw

### Attack timeline

| # | Time | Event | Source | Detection |
|---|---|---|---|---|
| 1 | 23:04:35 | A burst of A-queries with long unreadable subdomains — 13 data chunks (numbered 0–12) to the domain `exfil.lab`, from `192.168.100.20` to `192.168.100.24:53` (13 UDP datagrams) | agent on u3, found in `wazuh-archives-*`; DNS server query log (dnsmasq), Suricata | Recorded in `wazuh-archives-*`; **no alert fired in `wazuh-alerts-*`** — no rule exists for this attack type (MITRE ATT&CK T1071.004) |
| 2 | 23:04:35 | DNS server responses (`192.168.100.24:53` → `192.168.100.20`): `data.dns.rcode: REFUSED` to the same `exfil.lab` queries | agent on u3, `wazuh-archives-*`; Suricata DNS responses | Recorded in `wazuh-archives-*`; **no alert fired** — same missing rule |

No unrelated host noise (agent restarts, sudo, USB) was present in the ±1 minute
window around the burst.

## Event breakdown (attack-related only)

| Count | Source | Description |
|---|---|---|
| 13 | Suricata request | DNS A-queries (chunks 0–12) |
| 13 | Suricata response | REFUSED answers from `.24` |
| 13 | dnsmasq query log | the same queries, lowercased |

**Total: 39 events in the window, 0 alerts.** The whole incident was invisible
to the alerting layer.

## Key alert

There is none — and that is the point. No rule matches "suspicious outbound
traffic to unknown subdomains of unknown domains", so nothing fired. Finding
this required manual hunting in the archives.

One forensic detail worth keeping: tool register handling differs.

```
dnsmasq  -- 12.nk1nis9kzhnxswdcbwsxqzqlpirrxx8xaphwnujpb4k2eve2ba.exfil.lab  (lowercase)
Suricata -- 12.nK1Nis9KZhNXSwDCBWSxqzqLpIrrxX8xaPhWNUjPB4K2EVE2BA.exfil.lab  (as sent)
```

DNS is case-insensitive and tools normalize differently — a useful tell when
correlating two sources.

## Evidence

- [`iocs/wazuhe-arhive-18.11.2026(00.04.35).json`](iocs/wazuhe-arhive-18.11.2026(00.04.35).json) — 39 events (Suricata req/resp + dnsmasq) at the time of the suspected leak
- [`iocs/discover-screenshot.png`](iocs/discover-screenshot.png) — Discover view of the hunt
- [`iocs/dig-output.txt`](iocs/dig-output.txt) — attacker's own capture (chunks 0, 6, 12)

## Detection gaps (be honest)

- **No rule** for this behavior pattern. Recommendation: create detection for
  bursts of queries to random-looking subdomains of unknown/rare domains
  (fast, sequential, atypical) at severity ≥ 8.
- **Archives were disabled** at the time class of gaps was found (`logall_json`
  was off until SOC-2026-003); raw events only survive if a rule matches.
  Both gaps were fixed during this case — the lesson stands for other networks.

## Hunter queries

```
exfil.lab                                          # broad, text search
data.dns.queries.rrname : *exfil.lab*              # structured, Suricata only
```

The structured query returns parsed DNS fields (src, rcode, rrname) instead of
whole log lines — better for detection logic.

## MITRE ATT&CK Mapping

- **T1071.004 Application Layer Protocol: DNS** — presumably company data,
  encoded, exfiltrated over DNS from host `192.168.100.20` to `192.168.100.24:53`
