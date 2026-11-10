> Back to [case summary](README.md)

# Response & Hardening

## Immediate containment (lab reference)

```bash
# Lab only. In production — block at the perimeter firewall.
sudo iptables -A INPUT -p tcp -s 192.168.100.20 -j DROP
```

## Eradication & investigation

- Invalidate sessions / rotate secrets; force password resets.
- Scope review: all `/api/*` and `/ftp/*` requests from the source IP in access.log.
- Fix root causes: parameterized queries, disable directory listing, remove sensitive files from web root, stop relying on robots.txt for "security".

## Hardening / SOC Improvements

- Deploy a WAF (ModSecurity + OWASP CRS) — blocks SQLi payloads in phase 2.
- Ensure nginx `access_log` captures all vhosts and the Wazuh agent ships them reliably (no rotation gaps).
- Enable Suricata `etpro/exploit` SQLi rules if missing; alert on `..` sequences and ` OR 1=1` patterns in URIs/bodies.
- Consider application-level audit logging (who accessed admin APIs, from where) — JWT abuse is a blind spot for network IDS.
- Treat `robots.txt` entries as an attack-surface inventory: anything "disallowed" should be access-controlled, not just unlinked.
