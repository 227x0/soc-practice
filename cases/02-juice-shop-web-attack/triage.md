> Back to [case summary](README.md)

# L1 Analyst Triage

L1 never touches the target host — the job is to confirm the incident within 10–15 minutes and package the facts for L2.

1. **Aggregation.** All events from `192.168.100.20` → **one incident**, several stages (recon → enumeration → exploitation → exfiltration). Never file "three scans" as three incidents.
2. **Alert reading discipline.** The MSSQL-1433 alert means *probing*, not *open port*. Verify against other sources before writing "attacker found databases" in a report.
3. **False-positive check.** `192.168.100.20` is internal (Kali). In a real SOC, verify ownership in CMDB / authorized-scanner allowlists before recommending a block.
4. **Severity ladder.** Scan = informational. `200 OK` on a hidden resource = data leak. Successful login bypass = account takeover. Admin API dump = **critical, P1**.
5. **Visibility gap.** `GET /api/Users` with an admin Bearer token produced **no alert** in the SIEM. The nginx `access.log` line may have been shipped, but there is no rule that fires on "admin API call from a new source IP". Record the gap in the ticket — L1 cannot analyze what is not alerted.
6. **Verdict: true positive, P1 — confirmed web compromise with PII exfiltration.**

## L1 checklist

- [ ] Confirm the multi-stage kill-chain from one source IP.
- [ ] Read Suricata alerts carefully — distinguish probes from successful exploitation.
- [ ] Check source IP ownership in CMDB / authorized-scanner lists.
- [ ] Correlate Wazuh, Suricata, and application logs.
- [ ] Identify visibility gaps and explicitly document them.
- [ ] Set severity: successful web compromise with PII exfiltration = P1.
- [ ] Prepare a complete escalation package for L2.
