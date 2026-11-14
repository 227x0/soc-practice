# Lessons Learned — SOC-2026-003

- **Audit the stock ruleset before writing custom rules.** The "we have no SQLi detection" roadmap item was half-wrong: Wazuh ships rule 31164 for exactly that. The actual gap was *severity*, not coverage. Detection engineering starts with knowing what you already have. Run `wazuh-logtest` on representative attack lines before opening a text editor.

- **Config changes on agents and managers are production changes.** One bad `log_format` killed log collection on u3; one stray `</rules>` silently unloaded every rule on u1. Rituals that feel like bureaucracy (backup → config test → restart → reconnect/log check) exist because both failure modes are silent.

- **Build differences bite.** Wazuh docs list `auditd` as a valid log format; the 4.14.10 logcollector binary on this host disagrees. Test against the actual binary, not the documentation.

- **Shell hygiene is security hygiene.** `$(url)` expanded by an outer SSH shell, writing garbage into SIEM rules, is the same class of bug as unquoted variables in deletion scripts. Through nested SSH, prefer base64 payloads over heredocs.

- **Upgrade windows are case material.** What you installed, why, and how you verified it belongs in the same repo as attack cases. It's the "known good" baseline that makes every later anomaly meaningful.
