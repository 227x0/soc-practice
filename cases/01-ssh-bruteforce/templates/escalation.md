> L2 escalation ticket template. Copy, fill in, and send via your ticketing system.

```
[ESCALATION][P2] SSH brute-force SUCCESS on <TARGET_HOST> (<TARGET_IP>)
Rule: <WAZUH_RULE_ID> "<WAZUH_RULE_DESCRIPTION>" (L<RULE_LEVEL>)
Time: <UTC_TIMESTAMP>
Target: host <TARGET_HOST> / account "<ACCOUNT>"
Source: <SRC_IP> (<internal|external> — verify ownership in CMDB)
Evidence: <N> failed logins in <TIME_WINDOW>, then PAM/session opened
Context: <no prior alerts / prior alerts / known authorized scanner>
Requested:
  - block <SRC_IP> at firewall
  - force password reset for <ACCOUNT>
  - review post-auth activity: last, auth.log, bash_history
```
