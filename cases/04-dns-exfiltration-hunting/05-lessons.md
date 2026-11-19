> Back to [case summary](README.md)

# Lessons Learned

- **DNS tunneling traffic is readable once you know the shape.** A burst of
  long, unreadable, sequentially numbered subdomains to one domain is a script,
  not a user. Both telemetry sources told the same story from different angles.
- **13 chunks became 39 log events.** Every query produces a request, a
  response, and a resolver log line — 13 × 3 = 39. When correlating counts,
  always know how many artifacts each action generates per source.
- **Explaining suspicion is a skill.** "Suspicious" must be decomposable:
  volume per second, label entropy, domain novelty, sequential numbering,
  direction. If you cannot name the indicators, you cannot write the rule.
- **Tools normalize differently.** dnsmasq logged the queries lowercase,
  Suricata preserved the original case. Correlating sources means accounting
  for normalization, not assuming identical strings.
- **Alert ≠ event.** The entire incident produced zero alerts; it only existed
  in the archives. If nobody hunts where the alerts aren't, the attack wins.
