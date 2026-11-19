> Back to [case summary](README.md)

# L1 Analyst Triage

1. **Validation.** The pattern is clearly atypical: 13 requests within one
   second to unreadable subdomains of a domain unknown to the company. The
   sequential numbering 0–12 means a script, not a human — not typical behavior.
2. **False-positive check.** The source IP (`192.168.100.20`) has no persistent
   association with any company employee and is not listed in any company
   documentation as a corporate address or partner tool. (In this lab it is
   the Kali attack host; in production this is where the CMDB/asset inventory
   answer would go.)
3. **Enrichment.** Target (`192.168.100.24`): on this stand the host runs the
   Wazuh-managed services and dnsmasq acting as a local DNS server/forwarder on
   53/udp. In production, `.24`'s place would typically be an internal resolver,
   domain controller, or critical network node — the host role determines how
   alarming the fact of handling such queries (and the REFUSED status) is.

   **History (lookback):** within our sampled window, a single burst of 13
   chunks (0..12) to `exfil.lab` at 00:04:35 local (23:04:35 UTC) was recorded.
   For full enrichment a 24h lookback must be explicitly checked in the
   indexes/archives: prior DNS bursts from `.20`, other suspicious domains, or
   other anomalous activity.

   **Network context:** in the examined fragment the sender is a single host.
   One client = a point case; dozens = mass propagation.
4. **Severity.** **P2** — exfiltration is confirmed, but the volume is small,
   a single host is involved, and there are no signs of spread. (The detection
   gap itself — no rule, archives off — is a process finding, tracked
   separately; it does not by itself raise severity.)
5. **Verdict:** True positive, P2 — confirmed DNS exfiltration from an internal
   host; no malware evidence; scope limited to one host.

## L1 checklist

- [x] Malicious pattern confirmed from the event sequence
- [x] Source IP checked (asset inventory / CMDB / scanner allowlist)
- [x] Target host role checked (standard/service/privileged)
- [x] Severity set with justification
- [x] Escalation package complete (short ticket + evidence list)
- [ ] 24h host alert history reviewed — noted as a gap in enrichment
