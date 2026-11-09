> All IPs and accounts are fictional and used only in a home lab.

# IOCs

This folder contains artifacts from the case.

## Files

- `auth.log.fragment` — reconstructed fragment of `/var/log/auth.log` from the target host `u3`.
- `screenshots/` — placeholder for Wazuh SIEM screenshots and terminal evidence.

## Notes

- The `auth.log` fragment is reconstructed from the Wazuh alert timeline (`~37 alert events` followed by a successful PAM session).
- Real Wazuh screenshots must be exported from the lab SIEM and added to `screenshots/`.
