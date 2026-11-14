# Victim Telemetry Upgrade (u3 .24)

## auditd

```bash
sudo apt-get install -y auditd
sudo tee /etc/audit/rules.d/lab.rules << 'EOF'
-a always,exit -F arch=b64 -S execve,execveat -k lab_exec
-w /etc/passwd -p wa -k lab_identity
-w /etc/shadow -p wa -k lab_identity
-w /etc/nginx/ -p wa -k lab_nginx_cfg
-w /home/u3/.ssh -p wa -k lab_ssh_keys
-w /var/www -p wa -k lab_webroot
EOF
sudo service auditd restart
sudo auditctl -l    # verify rules are loaded
```

What this buys the Blue Team — every one of these now lands in `/var/log/audit/audit.log` and ships to the SIEM:

| key | catches |
|---|---|
| `lab_exec` | every executed command — post-exploitation timelines |
| `lab_identity` | user add / password tampering (`/etc/passwd`, `/etc/shadow`) |
| `lab_nginx_cfg` | reverse-proxy tampering (new routes to hide C2, etc.) |
| `lab_ssh_keys` | planted authorized_keys — persistence |
| `lab_webroot` | dropped webshells |

## Wazuh agent — shipping audit.log

Added to `/var/ossec/etc/ossec.conf`:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>
```

(Back up before editing; verify with `grep -c "</ossec_config>"` — must be 1. This host already had a copy-paste accident documented in `soc-lab/target-host.md` — the ritual exists for a reason.)

Then `sudo systemctl restart wazuh-agent`; confirm in `/var/ossec/logs/ossec.log`:

```
wazuh-agentd: INFO: (4102): Connected to the server ([192.168.100.22]:1514/tcp).
```

## Pitfall (hit during this upgrade)

Wazuh 4.14.10's `wazuh-logcollector` build on this host **rejects both `auditd` and `audit_log`** as `log_format` values:

```
wazuh-logcollector: ERROR: (1235): Invalid value for element 'log_format': auditd.
wazuh-logcollector: ERROR: (1235): Invalid value for element 'log_format': audit_log.
```

(the valid-token strings are absent from the binary at all). The accepted fallback is plain `syslog`: lines ship as raw text and are matched manager-side. If your build accepts `auditd`, prefer it — but test the config before restart; a broken `ossec.conf` takes the whole agent down (observed live: agent failed to start, service exited with error until fixed).

## Verification that telemetry flows

On u3:

```bash
sudo auditctl -a always,exit -F arch=b64 -S execve -k test_wakeup
/bin/echo hello-lab
sudo ausearch -k test_wakeup -ts recent | head
```

Then confirm the corresponding event reaches the SIEM (Security Events, agent `u3`, source audit). Rule **100200** (`lab_exec`, L10) fires on it once manager-side decoding matches.
