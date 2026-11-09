# Screenshots

This folder should contain evidence screenshots from the Wazuh SIEM and the target host.

## Required screenshots

1. **Wazuh Security Events dashboard** showing the alert burst around rule 40112.
2. **Alert detail view** for rule 40112 — full time, source IP, target user, and rule description.
3. **Timeline of failed attempts** — multiple rows of rule 5760 / 5557 leading up to the success.
4. **PAM session opened / closed** (rule 5501 / 5502) confirming the successful login.
5. **`/var/log/auth.log` terminal view** on the target host showing `Accepted password`.
6. **`last` output** showing the SSH session from `192.168.100.20`.

## Before uploading

- [ ] Verify no real personal email, domain, or public IP is visible.
- [ ] Blur or crop any unrelated hosts / users.
- [ ] Ensure hostnames, IPs, and account names match the lab only (`192.168.100.20`, `192.168.100.24`, `u3`).

> I do not have access to your lab environment, so I cannot create real Wazuh screenshots. Please export them from your SIEM and drop the PNG/JPG files here.
