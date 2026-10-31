# ☕ Espresso, Firmware Reverse Engineering Writeup

> **Challenge name:** Espresso  
> **Category:** Reverse Engineering / Firmware  
> **Given file:** `firmware.bin`  
> **Goal:** Understand what the leaked Espresso firmware does and recover the flag.

---

## 1. Executive Summary

The challenge gives a raw firmware dump named `firmware.bin`. At first, the flag is not visible through normal string searching because it is stored in encrypted form inside the firmware's read-only data section.

The firmware is an **ESP32 firmware image**. The useful application image starts at flash offset `0x10000`. After inspecting the ESP32 image layout and reversing the application code, we find a small routine that decrypts a byte array using a single-byte XOR key.

The encrypted flag bytes are stored at virtual address:

```text
0x3f407688
```

The code XORs every encrypted byte with:

```text
0x42
```

After decrypting the byte array, the flag becomes:

```text
HTB{3mul4ting_hw_is_s0_c00l!!!}
```

---

## 2. Files Provided

```text
firmware.bin
```

Basic file information:

```bash
ls -lh firmware.bin
file firmware.bin
```

Observed result:

```text
-rw-r--r-- 1 user user 4.0M firmware.bin
firmware.bin: data
```

The `file` command only says `data`, which means it does not directly recognize the format. That is normal for many raw firmware images.

---

## 3. Initial Recon

The first thing to do with unknown firmware is basic static triage.

Useful commands:

```bash
strings -a -n 5 firmware.bin | head
strings -a -n 5 firmware.bin | grep -iE "flag|clone|emulat|espresso|HTB"
```

Important strings discovered:

```text
espresso
flag did not generate correctly
It seems you are running the firmware on cloned hadware
Buy the real hardware, or perhaps try to emulate it. ;)
```

These strings tell us several useful things:

1. The binary probably contains custom challenge logic.
2. The flag is generated at runtime.
3. There is likely an anti-clone or anti-emulation check.
4. Simply running the firmware may fail unless the hardware behavior is emulated correctly.
5. The flag is probably not stored as plaintext.

At this stage, searching for `HTB` does not work:

```bash
strings -a firmware.bin | grep HTB
```

No plaintext flag appears.

---

## 4. Identifying the Firmware Format

A raw ESP32 firmware commonly contains several important flash offsets:

| Offset | Purpose |
|---:|---|
| `0x1000` | Second-stage bootloader |
| `0x8000` | Partition table |
| `0x10000` | Factory application image |

We can confirm this using a small Python parser.

```python
from pathlib import Path

b = Path("firmware.bin").read_bytes()

print("Firmware size:", hex(len(b)))

for off in range(0, 0x20000, 0x1000):
    if b[off] == 0xE9:
        seg_count = b[off + 1]
        entry = int.from_bytes(b[off + 4:off + 8], "little")
        print(f"ESP image magic at {off:#x}, segments={seg_count}, entry={entry:#x}")
```

Output:

```text
Firmware size: 0x400000
ESP image magic at 0x1000, segments=3, entry=0x40080644
ESP image magic at 0x10000, segments=6, entry=0x400814ac
```

This confirms that the file contains ESP images. The image at `0x1000` is the bootloader. The image at `0x10000` is the main application.

---

## 5. Parsing the Partition Table

The partition table begins at `0x8000`. ESP32 partition entries are 32 bytes long and normally start with the magic bytes:

```text
AA 50
```

A quick parser gives us the important partitions:

```python
from pathlib import Path

b = Path("firmware.bin").read_bytes()
pt = 0x8000

for i in range(32):
    off = pt + i * 32
    ent = b[off:off + 32]

    if ent[:2] == b"\xff\xff":
        break

    if ent[:2] != b"\xaa\x50":
        break

    typ = ent[2]
    sub = ent[3]
    part_off = int.from_bytes(ent[4:8], "little")
    size = int.from_bytes(ent[8:12], "little")
    label = ent[12:28].split(b"\x00")[0].decode(errors="ignore")

    print(i, label, hex(typ), hex(sub), hex(part_off), hex(size))
```

Output:

```text
0 nvs      0x1 0x2 0x9000  0x6000
1 phy_init 0x1 0x1 0xf000  0x1000
2 factory  0x0 0x0 0x10000 0x100000
```

The main target is therefore:

```text
factory app at offset 0x10000
```

---

## 6. Understanding ESP32 App Segments

The application image has several loadable segments. Each segment has:

1. A virtual load address.
2. A segment size.
3. Segment data stored in the firmware file.

A small parser shows the app layout:

```python
from pathlib import Path

b = Path("firmware.bin").read_bytes()
base = 0x10000

seg_count = b[base + 1]
entry = int.from_bytes(b[base + 4:base + 8], "little")

print(f"App entry: {entry:#x}")
print(f"Segments: {seg_count}")

o = base + 24
for i in range(seg_count):
    load = int.from_bytes(b[o:o + 4], "little")
    size = int.from_bytes(b[o + 4:o + 8], "little")
    data_off = o + 8

    print(
        f"segment {i}: "
        f"vaddr={load:#x}, size={size:#x}, "
        f"file={data_off:#x}-{data_off + size:#x}"
    )

    o = data_off + size
```

Important output:

```text
App entry: 0x400814ac
Segments: 6
segment 0: vaddr=0x3f400020, size=0x8b68, file=0x10020-0x18b88
segment 1: vaddr=0x3ffb0000, size=0x2a6c, file=0x18b90-0x1b5fc
segment 2: vaddr=0x40080000, size=0x4a14, file=0x1b604-0x20018
segment 3: vaddr=0x400d0020, size=0xc7b8, file=0x20020-0x2c7d8
segment 4: vaddr=0x40084a14, size=0x60f4, file=0x2c7e0-0x328d4
segment 5: vaddr=0x50000000, size=0x28,   file=0x328dc-0x32904
```

The important parts are:

| Segment | Virtual address | Meaning |
|---:|---:|---|
| Segment 0 | `0x3f400020` | Usually read-only data, strings, constants |
| Segment 3 | `0x400d0020` | Application code |

So we expect encrypted data near `0x3f40....` and code near `0x400d....`.

---

## 7. The Important Runtime Clue

The firmware contains this message:

```text
Buy the real hardware, or perhaps try to emulate it. ;)
```

That message is intentionally misleading and helpful at the same time.

It hints that the firmware checks something hardware-related. In ESP32 firmware, this often means reading values from:

- eFuse
- MAC address registers
- flash chip information
- chip revision data
- unique hardware identifiers

However, for this challenge, full hardware emulation is not actually necessary. Static reversing is enough because the encrypted flag and the decryption routine are both present inside the firmware.

---

## 8. Reversing the Decryption Logic

The application code contains a small loop that reads encrypted bytes from `.rodata`, XORs them with a constant, and writes the decrypted bytes into an output buffer.

The key instruction sequence is:

```asm
400d5cbd: L32R   a9, [400d07d8] = 3f407688
400d5cc2: L8UI   a9, a9, 0
400d5cc7: MOVI.N a11, 66
400d5cc9: XOR    a9, a9, a11
400d5ccc: S8I    a9, a10, 0
```

Let us translate this into simple English.

| Instruction | Meaning |
|---|---|
| `L32R a9, ... = 3f407688` | Load the address of the encrypted byte array into register `a9` |
| `L8UI a9, a9, 0` | Read one unsigned byte from that encrypted array |
| `MOVI.N a11, 66` | Put decimal `66` into register `a11` |
| `XOR a9, a9, a11` | XOR the encrypted byte with `66`, which is `0x42` |
| `S8I a9, a10, 0` | Store the decrypted byte into the output buffer |

Decimal `66` is hexadecimal:

```text
66 decimal = 0x42 hex
```

So the decryption is:

```python
plain_byte = encrypted_byte ^ 0x42
```

This is a classic single-byte XOR decryption loop.

---

## 9. Locating the Encrypted Flag

The decryption routine loads encrypted bytes from:

```text
0x3f407688
```

From the app segment table:

```text
segment 0 virtual base = 0x3f400020
segment 0 file range   = 0x10020 - 0x18b88
```

To convert a virtual address to a firmware file offset:

```text
file_offset = segment_file_offset + (target_virtual_address - segment_virtual_base)
```

Apply the values:

```text
file_offset = 0x10020 + (0x3f407688 - 0x3f400020)
file_offset = 0x17688
```

So the encrypted flag begins at file offset:

```text
0x17688
```

Encrypted bytes:

```text
0a160039712f372e76362b2c251d2a351d2b311d31721d2172722e6363633f
```

---

## 10. Decrypting the Flag

Now we can decrypt the bytes directly with Python.

```python
enc = bytes.fromhex(
    "0a160039712f372e76362b2c251d2a351d2b311d31721d2172722e6363633f"
)

key = 0x42
flag = bytes(b ^ key for b in enc)

print(flag.decode())
```

Output:

```text
HTB{3mul4ting_hw_is_s0_c00l!!!}
```

---

## 11. Final Flag

```text
HTB{3mul4ting_hw_is_s0_c00l!!!}
```

For a public writeup, remove or redact this section if the platform or event rules require flag redaction.

---

## 12. Complete Solver Script

This script extracts the encrypted bytes from the firmware at the correct offset and decrypts them.

```python
#!/usr/bin/env python3
from pathlib import Path

fw = Path("firmware.bin").read_bytes()

# Encrypted flag location recovered from reversing:
# virtual address: 0x3f407688
# file offset:     0x17688
enc_off = 0x17688
enc_len = 31
key = 0x42

enc = fw[enc_off:enc_off + enc_len]
flag = bytes(b ^ key for b in enc)

print(flag.decode())
```

Run it:

```bash
python3 solve.py
```

Expected output:

```text
HTB{3mul4ting_hw_is_s0_c00l!!!}
```

---

## 13. Why `strings` Alone Failed

A normal string search failed because the flag was not stored like this:

```text
HTB{...}
```

Instead, every byte was XOR-encrypted with `0x42`.

Example:

| Plain byte | ASCII | XOR key | Encrypted byte |
|---:|---|---:|---:|
| `0x48` | `H` | `0x42` | `0x0a` |
| `0x54` | `T` | `0x42` | `0x16` |
| `0x42` | `B` | `0x42` | `0x00` |
| `0x7b` | `{` | `0x42` | `0x39` |

So the flag starts encrypted as:

```text
0a 16 00 39 ...
```

That does not look like readable text, so `strings` ignores it.

---

## 14. Visual Flow

```text
firmware.bin
    |
    v
Identify ESP32 image layout
    |
    +--> bootloader at 0x1000
    +--> partition table at 0x8000
    +--> factory app at 0x10000
                    |
                    v
              Parse app segments
                    |
                    +--> .rodata at 0x3f400020
                    +--> code at 0x400d0020
                    |
                    v
       Find suspicious anti-clone strings
                    |
                    v
       Reverse nearby flag generation logic
                    |
                    v
       Encrypted flag at 0x3f407688
                    |
                    v
       XOR each byte with 0x42
                    |
                    v
       HTB{3mul4ting_hw_is_s0_c00l!!!}
```

---

# Knowledge Gaps Filled

This section explains the concepts needed to properly understand the challenge.

---

## A. What Is Firmware?

Firmware is software designed to run on embedded hardware, such as:

- routers
- IoT devices
- microcontrollers
- smart appliances
- development boards

Unlike a normal Linux executable, firmware is often a raw flash image. It may contain:

- bootloader code
- partition tables
- application code
- configuration data
- filesystem data
- calibration data

That is why `file firmware.bin` may only show `data`.

---

## B. What Is ESP32?

ESP32 is a popular microcontroller family made by Espressif. It is widely used in IoT devices.

Important ESP32 reversing facts:

| Topic | Explanation |
|---|---|
| CPU architecture | Xtensa LX6 on classic ESP32 |
| Common firmware magic | `0xE9` |
| Bootloader offset | Usually `0x1000` |
| Partition table offset | Usually `0x8000` |
| App offset | Often `0x10000` |
| Code addresses | Often around `0x400d....` |
| Read-only data | Often around `0x3f40....` |

In this challenge, these patterns matched perfectly.

---

## C. What Is a Partition Table?

A partition table tells the chip where each section of flash memory is located.

Example from this firmware:

```text
nvs      -> 0x9000
phy_init -> 0xf000
factory  -> 0x10000
```

The `factory` partition contains the real application we need to reverse.

---

## D. What Is `.rodata`?

`.rodata` means **read-only data**.

It usually contains:

- constant strings
- lookup tables
- encrypted blobs
- format strings
- constant byte arrays

The encrypted flag was stored in this kind of region.

---

## E. What Is XOR Encryption?

XOR is a bitwise operation. It is commonly used in CTF challenges because it is simple and reversible.

Rule:

```text
encrypted = plaintext ^ key
plaintext = encrypted ^ key
```

The same operation encrypts and decrypts.

Example:

```text
'H'       = 0x48
key       = 0x42
encrypted = 0x48 ^ 0x42 = 0x0a
```

To recover it:

```text
0x0a ^ 0x42 = 0x48 = 'H'
```

That is why a single Python one-liner can decrypt the whole flag once the key is known.

---

## F. What Is `L32R` in Xtensa?

`L32R` is an Xtensa instruction that loads a 32-bit value from a nearby literal pool.

In this challenge:

```asm
L32R a9, [400d07d8] = 3f407688
```

This means:

```text
register a9 = 0x3f407688
```

That address points to the encrypted flag bytes.

---

## G. Why the Anti-Emulation Message Matters

The message:

```text
Buy the real hardware, or perhaps try to emulate it. ;)
```

suggests the firmware may behave differently on real hardware versus an emulator.

That is a common embedded challenge trick. The firmware may check hardware-specific values such as:

- MAC address
- eFuse registers
- flash chip ID
- chip revision

However, the challenge can be solved statically because the decryption routine and encrypted bytes are both inside the firmware.

---

## H. Common Mistakes in This Challenge

### Mistake 1: Only using `strings`

`strings` is useful, but it only finds plaintext. It cannot recover encrypted byte arrays.

### Mistake 2: Reversing the bootloader instead of the app

The firmware contains a bootloader at `0x1000`, but the custom challenge logic is in the factory app at `0x10000`.

### Mistake 3: Ignoring virtual addresses

The code references virtual address `0x3f407688`, not direct file offset `0x3f407688`.

You must map virtual addresses back to firmware file offsets using the segment table.

### Mistake 4: Overcomplicating the emulation hint

The hint talks about emulation, but the flag can be recovered without fully emulating the firmware.

---

## 15. Clean Methodology for Similar Firmware Challenges

Use this checklist for future firmware reversing challenges.

```text
1. Run file, strings, and entropy checks.
2. Look for known firmware magic bytes.
3. Identify architecture and platform.
4. Locate partition tables or filesystem sections.
5. Extract the application region.
6. Load the app into a disassembler with the right architecture.
7. Search for suspicious strings.
8. Cross-reference those strings in code.
9. Look for decode, decrypt, compare, or print routines.
10. Rebuild the algorithm in Python.
11. Verify the output format, for HTB usually HTB{...}.
```

---

## 16. Final Takeaway

The main lesson from Espresso is that firmware challenges often require understanding both the file format and the CPU code. The firmware tried to hide the flag behind a hardware or emulation-themed message, but the actual protection was a simple XOR routine.

Once the ESP32 image layout was understood, the solution became straightforward:

```text
Find encrypted bytes -> identify XOR key -> decrypt -> recover flag
```

Final flag:

```text
HTB{3mul4ting_hw_is_s0_c00l!!!}
```
