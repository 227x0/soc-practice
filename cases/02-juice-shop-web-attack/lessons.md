> Back to [case summary](README.md)

# Lessons Learned

- **IDS catches scanners and payloads, not sessions obtained through auth bypass.** The final exfiltration stage rode on a valid admin token and was invisible to Suricata. Behavior correlation (new IP → admin-scope API burst within minutes of "login") is the only network-side defense.
- **Alert ≠ fact.** A "Suspicious inbound to MSSQL port" alert reflects a single probe packet; the port was closed. Analysts must correlate before reporting impact.
- **Aggregation is a core L1 skill.** One incident, many alerts; the timeline — not the alert count — tells the story.
- **Slow scans evade rate-based rules** (`-T2` produced sparse alerts over minutes). They are caught by source-IP correlation over a time window, exactly how this exercise was detected.
- **Visibility gaps belong in L1 reports.** "We cannot see X" is a finding, not an excuse — it drives engineering work (log coverage, rulesets, app audit trails).
