# SIEM Package (u1 .22), applied and verified

## Discovery: stock ruleset already detects SQLi

Before writing custom rules, a `wazuh-logtest` run against a synthetic SQLi access-log line revealed that Wazuh's stock ruleset already matches classic patterns:

```
**Phase 3: Completed filtering (rules).
	id: '31164'
	level: '6'
	description: 'SQL injection attempt.'
	mitre.id: '['T1055', 'T1190']'
**Alert to be generated.
```

So the detection-engineering roadmap item from SOC-2026-002 ("custom Wazuh rules for SQLi") was reframed: what the lab actually needs is not *new* pattern matching but an **escalation**: stock rule 31164 sits at level 6, below the L1 triage threshold.

## Rules applied

`/var/ossec/etc/rules/local_rules.xml` inside the manager container (persistent volume; backup at `local_rules.xml.bak` from the upgrade window). Full content in [`iocs/local_rules.xml`](iocs/local_rules.xml).

| rule | level | logic |
|---|---|---|
| **100100** | 12 | `<if_sid>31164</if_sid>`: escalates stock SQLi detection to L12 with srcip/url in the description, MITRE T1190 |
| **100200** | 10 | `audit.key = lab_exec`: groups auditd process-execution events from the u3 telemetry upgrade |

### Verified with wazuh-logtest

Input (synthetic nginx access line):

```
192.168.100.20 - - [10/Nov/2026:06:10:00 +0000] "GET /rest/user/login?email=%27%20OR%201=1-- HTTP/1.1" 200 512 "-" "curl/7.88"
```

Output:

```
	description: 'SOC-LAB: SQLi pattern on web target escalated from stock rule 31164
	(src: 192.168.100.20, url: /rest/user/login?email=%27%20OR%201=1--)'
**Alert to be generated.
```

## Pitfall (hit during this upgrade)

Two, both self-inflicted and worth remembering:

1. **`$(url)` inside a heredoc through nested SSH** gets expanded by the outer shell (`url: command not found`) and writes garbage into the target file. When pushing config through multiple shell layers, use base64 payloads, not heredocs.
2. **`</rules>` is not a real tag.** `local_rules.xml` has no root element. Appending a closing `</rules>` makes `wazuh-analysisd` fail with `XMLERR: Element not opened` and **no rules load at all** (manager boots, everything silently unprotected). The stock file simply ends after the last `</group>`.

## Analyst utilities on u1

| tool | purpose | status |
|---|---|---|
| `jq` | eve.json / API surgery | ✅ installed |
| `yara` | malware triage on captured samples | ✅ installed |
| `sigma-cli` | portable Sigma rules, convert to Wazuh backend | ⏳ via pipx (install retried after `apt update`) |

All chosen to fit the 30 GB disk where TheHive/MISP do not (yet).
