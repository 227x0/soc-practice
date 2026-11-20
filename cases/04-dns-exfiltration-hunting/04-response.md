> Back to [case summary](README.md)

# Response & Hardening

Suspicious domain **exfil.lab** — pattern consistent with DNS tunneling.

## Immediate containment

Block / sinkhole the domain **exfil.lab** at DNS level (dnsmasq / resolver) and
block outbound 53/udp from hosts, except allowed company internal resolvers.

## Eradication & investigation

- Full 24-hour log review for the same pattern (other senders, other domains,
  similar bursts).
- On host `192.168.100.20`: identify which script/process generated the requests
  to `exfil.lab`.
- Decode the chunk sequence from the subdomains to establish what data was taken.

## Hardening

- **Architecture:** block direct outbound 53/udp that bypasses the corporate
  resolver; implement RPZ (Response Policy Zone) / sinkhole for suspicious
  domains; restrict length and frequency of non-standard DNS queries.
- **Detection:** add a Wazuh/SIEM rule for anomalous DNS traffic — excessively
  long labels/subdomains, bursts of queries to a single domain in a short
  window, high REFUSED/NXDOMAIN ratio, high domain-name entropy.

## What L1 learned from this case

In `wazuh-archives-*`, one event can appear duplicated across several sources
(Suricata request/response + the matching `dnsmasq` line), and the time shown
in the Discover UI can differ from the event's own timestamp because of
timezone settings. Always correlate deliberately, never assume.
