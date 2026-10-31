# 🧩 Hack The Box Challenge Writeup: Maze

> **Challenge:** Maze  
> **Category:** Reverse Engineering / Forensics / Crypto  
> **Difficulty Style:** Multi-stage puzzle  
> **Goal:** Escape the maze and recover the valid HTB flag.

---

## 📌 Executive Summary

The challenge gives three important files:

| File | Purpose |
|---|---|
| `maze.exe` | A Windows executable packed with PyInstaller. It hides Python logic and the ZIP password. |
| `enc_maze.zip` | An AES-encrypted ZIP archive containing an obfuscated Linux ELF binary named `maze`. |
| `maze.png` | A hint carrier. Specific bytes inside the image generate the random seed needed for deobfuscation. |

The full solve chain is:

```text
maze.exe
  ↓ PyInstaller extraction
Python bytecode / obfuscated route logic
  ↓ recover ZIP password + PNG byte offsets
enc_maze.zip
  ↓ AES ZIP extraction
obfuscated maze file
  ↓ random-seed deobfuscation
valid Linux ELF binary
  ↓ reverse validation logic
adjacent-character sum constraints
  ↓ solve constraints
final HTB flag
```

The challenge is not solved by only running the files. It is solved by reversing each stage and connecting the clues.

---

## 🗂️ Files Provided

```text
maze.exe
enc_maze.zip
maze.png
```

Initial identification:

```bash
file maze.exe enc_maze.zip maze.png
```

Expected result:

```text
maze.exe:     PE32+ executable for MS Windows, x86-64
enc_maze.zip: Zip archive data, AES Encrypted
maze.png:     PNG image data, 512 x 512, RGBA
```

The important observations are:

1. `maze.exe` is a Windows executable, but the challenge still contains Python artifacts.
2. `enc_maze.zip` is AES encrypted, so normal `unzip` may fail.
3. `maze.png` is not just decoration. It contains byte-level hint data.

---

## 🧠 Required Knowledge Before Solving

This challenge expects several small skills from different areas.

### 1. PyInstaller Basics

PyInstaller packages Python scripts into Windows or Linux executables. The executable usually contains:

```text
Python interpreter/runtime files
.pyc bytecode files
PYZ archive
bootloader code
embedded Python modules
```

So even if the file is `.exe`, it may still be a Python program internally.

### 2. Python Bytecode / `.pyc`

Python source code can be compiled into bytecode. If decompilation fails, strings and constants inside the bytecode are still useful.

In this challenge, the password and route logic are recoverable from Python bytecode constants.

### 3. AES ZIP Encryption

Some ZIP files use WinZip AES encryption. Standard tools may not support it properly.

Useful Python library:

```bash
python3 -m pip install pyzipper
```

### 4. ELF Reversing

After deobfuscation, the extracted file becomes a Linux ELF executable. Basic ELF reversing requires:

```bash
file
strings
readelf
objdump
chmod +x
```

### 5. Constraint Solving

The final binary does not directly store the flag. Instead, it checks mathematical constraints:

```text
flag[i-1] + flag[i] + flag[i+1] == target[i]
```

This can be solved with simple Python recursion once the first characters are known.

---

## 🔎 Stage 1: Inspecting `maze.exe`

First, check for PyInstaller indicators:

```bash
strings -a maze.exe | grep -Ei "pyi|pyinstaller|python|pyz"
```

Useful indicators:

```text
pyi-contents-directory
PYZ-00.pyz
pyiboot01_bootstrap
python38.dll
PyInstaller
```

This confirms that `maze.exe` is a PyInstaller-packed Python application.

---

## 📦 Stage 2: Extracting the PyInstaller Executable

Use `pyinstxtractor.py`:

```bash
python3 pyinstxtractor.py maze.exe
```

Typical output directory:

```text
maze.exe_extracted/
```

Important recovered components:

```text
PYZ-00.pyz
pyiboot01_bootstrap
maze.pyc / maze bytecode
obf_path bytecode
```

If decompilation works, use a Python decompiler such as `pycdc` or `uncompyle6`.

If decompilation does not work cleanly, use `strings` and bytecode constants:

```bash
strings -a maze.exe_extracted/* | grep -E "Y0u|maze|path|dead|zip"
```

---

## 🧾 Stage 3: Recovering the ZIP Password

From the extracted Python logic, the program references the encrypted archive:

```python
ZIPFILE = "enc_maze.zip"
```

The hidden password is stored as a string constant:

```text
Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH
```

The relevant Python logic is effectively:

```python
import pyzipper

ZIPFILE = "enc_maze.zip"

password = b"Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH"

with pyzipper.AESZipFile(ZIPFILE) as zf:
    zf.pwd = password
    zf.extractall()
```

This extracts a file named:

```text
maze
```

At this point, the extracted `maze` file is still not a valid ELF binary. It is intentionally obfuscated.

---

## 🖼️ Stage 4: Understanding the Role of `maze.png`

The obfuscated route logic contains this file name:

```python
index_file = "maze.png"
```

It then reads specific byte offsets from the image:

```python
index = open("maze.png", "rb").read()
seed_value = index[4817] + index[2624] + index[2640] + index[2720]
```

Calculating those bytes:

```python
from pathlib import Path

b = Path("maze.png").read_bytes()
offsets = [4817, 2624, 2640, 2720]
values = [b[i] for i in offsets]

print(values)
print(sum(values))
```

Output:

```text
[221, 96, 53, 123]
493
```

So the random seed is:

```python
seed(493)
```

The script also hints at this exact random pattern:

```python
seed(493)
for i in range(300):
    randint(32,125)
```

### Key Point

The PNG is not hiding a visible QR code or text message. The important clue is stored in raw byte positions.

This is a common CTF trick: the image is used as a byte container.

---

## 🔐 Stage 5: Extracting and Deobfuscating the ZIP Content

After decrypting the ZIP, we get a file named `maze`, but it is still damaged/obfuscated.

A quick check shows the first bytes are close to an ELF header but not correct:

```bash
file maze
```

It reports data instead of a valid executable.

A valid ELF should begin with:

```text
7f 45 4c 46
```

which is:

```text
. E L F
```

The extracted file starts with a corrupted first byte.

The deobfuscation logic is:

```python
from random import seed, randint

seed(493)
key = [randint(32, 125) for _ in range(300)]

for i in range(0, len(data), 10):
    data[i] = (data[i] + 80) % 256

for i in range(0, len(data), 10):
    data[i] = (data[i] ^ key[i % len(key)]) % 256
```

Only every 10th byte is modified.

---

## 🛠️ Full Decryption + Deobfuscation Script

Save this as `solve_stage1.py`:

```python
from pathlib import Path
from random import seed, randint
import pyzipper

ZIP_FILE = "enc_maze.zip"
ZIP_PASSWORD = b"Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH"
PNG_FILE = "maze.png"
OUT_FILE = "dec_maze"

# Step 1: recover seed from maze.png
png = Path(PNG_FILE).read_bytes()
offsets = [4817, 2624, 2640, 2720]
seed_value = sum(png[i] for i in offsets)
print(f"[+] Seed recovered from PNG: {seed_value}")

# Step 2: decrypt AES ZIP
with pyzipper.AESZipFile(ZIP_FILE) as zf:
    zf.pwd = ZIP_PASSWORD
    raw = zf.read("maze")

print(f"[+] Extracted encrypted maze payload: {len(raw)} bytes")

# Step 3: generate key stream
seed(seed_value)
key = [randint(32, 125) for _ in range(300)]

# Step 4: deobfuscate every 10th byte
data = bytearray(raw)

for i in range(0, len(data), 10):
    data[i] = (data[i] + 80) % 256

for i in range(0, len(data), 10):
    data[i] = (data[i] ^ key[i % len(key)]) % 256

# Step 5: write final ELF
Path(OUT_FILE).write_bytes(data)
print(f"[+] Wrote deobfuscated ELF: {OUT_FILE}")
print(f"[+] Header: {data[:4].hex()}  expected: 7f454c46")
```

Run it:

```bash
python3 solve_stage1.py
file dec_maze
chmod +x dec_maze
```

Expected result:

```text
dec_maze: ELF 64-bit LSB pie executable, x86-64
```

Now we have reached the real binary maze.

---

## 🧪 Stage 6: Testing the Deobfuscated ELF

Run it:

```bash
./dec_maze
```

Inputting a wrong value gives:

```text
You're going deeper into the maze...
```

A correct input prints:

```text
Well done for escaping the maze...
```

However, this binary has an important weakness: it does not strictly check the full flag length.

For example, some short prefixes may trigger the success message, but they are not valid HTB flags.

So we must reverse the full validation logic, not trust the success string alone.

---

## 🧬 Stage 7: Reversing the ELF Validation Logic

Start with strings:

```bash
strings -a dec_maze
```

Important strings:

```text
Well done for escaping the maze...
You're going deeper into the maze...
```

Disassemble with objdump:

```bash
objdump -d -Mintel dec_maze | less
```

The important logic does the following:

1. Reads input with `fgets`.
2. Removes the newline using `strcspn`.
3. Checks that the first three characters are:

```text
H T B
```

4. Loops over the input and checks sums of three adjacent characters.

The core check is equivalent to:

```c
if (input[i - 1] + input[i] + input[i + 1] != targets[i - 1]) {
    fail();
}
```

The target array is stored in `.rodata`.

Extracted target values:

```python
targets = [
    222, 273, 308, 290, 254, 230, 271, 232, 254,
    260, 279, 210, 232, 273, 325, 303, 264, 217,
    221, 204, 263, 215, 258, 230, 283, 237, 268,
    259, 287, 224, 219, 193, 192, 135, 117, 191
]
```

There are 36 target values.

Since each target checks three adjacent characters, the intended flag length is:

```text
36 + 2 = 38 characters
```

---

## 🧮 Stage 8: Solving the Character Constraints

We know the first two characters:

```text
H T
```

The first target is:

```text
flag[0] + flag[1] + flag[2] = 222
```

So:

```text
'H' + 'T' + flag[2] = 222
72 + 84 + flag[2] = 222
flag[2] = 66
```

ASCII `66` is:

```text
B
```

That gives:

```text
HTB
```

Then the same idea continues:

```text
flag[i + 1] = target[i - 1] - flag[i - 1] - flag[i]
```

So the entire flag can be recovered one character at a time.

---

## 🧰 Final Constraint Solver

Save this as `solve_flag.py`:

```python
targets = [
    222, 273, 308, 290, 254, 230, 271, 232, 254,
    260, 279, 210, 232, 273, 325, 303, 264, 217,
    221, 204, 263, 215, 258, 230, 283, 237, 268,
    259, 287, 224, 219, 193, 192, 135, 117, 191
]

# Known HTB prefix starts with H and T.
chars = [ord("H"), ord("T")]

for target in targets:
    next_char = target - chars[-1] - chars[-2]
    chars.append(next_char)

flag = "".join(chr(c) for c in chars)

print(flag)
print(f"Length: {len(flag)}")

# Sanity checks
assert flag.startswith("HTB{")
assert flag.endswith("}")
assert len(flag) == 38

# Verify every adjacent triple sum
for i, target in enumerate(targets):
    assert ord(flag[i]) + ord(flag[i + 1]) + ord(flag[i + 2]) == target

print("[+] All constraints passed")
```

Run:

```bash
python3 solve_flag.py
```

---

## 🏁 Final Flag

> Publishing note: remove or redact this section before posting a public HTB writeup if the challenge is still active.

<details>
<summary>Click to reveal private solved flag</summary>

```text
HTB{w0W_Y0u_C0uld_E5c4p3_Th1s_M4Z33!!}
```

</details>

---

## 🧩 Why the Challenge Is Called “Maze”

The name fits because the challenge creates several false or indirect paths:

| Maze Layer | What It Tries to Do |
|---|---|
| `maze.exe` | Looks like a normal Windows binary but is actually PyInstaller Python. |
| Obfuscated Python route | Gives misleading path messages and exits early. |
| `enc_maze.zip` | Blocks access with AES ZIP encryption. |
| `maze.png` | Looks like a normal image but contains byte offsets for the seed. |
| Extracted `maze` file | Looks broken until every 10th byte is repaired. |
| Final ELF | Accepts partial prefixes, so success output alone is misleading. |

The real path is found by reversing the structure, not by blindly executing files.

---

## ⚠️ Important Pitfalls

### Pitfall 1: Trusting Program Output Too Early

The final ELF can print the success message for short valid prefixes because it lacks a strict length check.

Do not stop at:

```text
Well done for escaping the maze...
```

You still need the full valid HTB flag.

### Pitfall 2: Ignoring the PNG

The image is not just a thumbnail or theme asset. The extracted Python logic directly reads bytes from it.

### Pitfall 3: Using Normal `unzip`

The archive uses AES encryption. If normal `unzip` fails, use `pyzipper`.

### Pitfall 4: Assuming the Extracted ZIP File Is the Final Binary

The extracted `maze` file is still obfuscated. You must repair it before reversing.

### Pitfall 5: Missing the Every-10th-Byte Pattern

Only positions `0, 10, 20, 30, ...` are changed.

That is why the file looks almost like an ELF but still fails initially.

---

## 📚 Knowledge Gaps to Fill

Use this checklist to strengthen the concepts used in this challenge.

### Reverse Engineering

- [ ] Learn how `file`, `strings`, `readelf`, and `objdump` work.
- [ ] Understand ELF magic bytes: `7f 45 4c 46`.
- [ ] Learn how input validation appears in assembly.
- [ ] Practice spotting `fgets`, `strlen`, `strcspn`, and `puts` in disassembly.

### Python Reversing

- [ ] Understand how PyInstaller bundles Python programs.
- [ ] Learn how to extract PyInstaller executables with `pyinstxtractor.py`.
- [ ] Learn the difference between `.py`, `.pyc`, and `PYZ` files.
- [ ] Practice reading Python constants when decompilation fails.

### Crypto / Encoding

- [ ] Understand AES-encrypted ZIP archives.
- [ ] Learn why `pyzipper` is useful for WinZip AES.
- [ ] Understand XOR operations.
- [ ] Understand modulo byte arithmetic: `% 256`.

### Scripting

- [ ] Practice writing small Python scripts for binary patching.
- [ ] Practice reading raw bytes from files.
- [ ] Practice generating deterministic random values with `seed()`.
- [ ] Practice solving simple recurrence constraints.

---

## 🧠 Concepts Explained Simply

### What is PyInstaller?

PyInstaller takes Python code and packs it into an executable. The `.exe` still contains Python bytecode inside it. That is why extracting it reveals Python logic.

### What is a Magic Header?

A magic header is a fixed byte pattern at the start of a file. Linux ELF files begin with:

```text
7f 45 4c 46
```

If the header is corrupted, Linux will not recognize it as a valid executable.

### What is XOR?

XOR is a reversible byte operation.

If:

```text
A XOR B = C
```

Then:

```text
C XOR B = A
```

That makes XOR common in CTF obfuscation.

### What does `seed()` do?

Random numbers in programming are often pseudo-random. If the same seed is used, the same random sequence is generated every time.

Example:

```python
from random import seed, randint

seed(493)
print(randint(32, 125))
```

This always gives the same first value.

### Why did the sum constraints work?

The binary checks groups of three adjacent characters. Once we know the first two characters, each next character becomes forced.

Example:

```text
known: H, T
sum:   H + T + ? = 222
answer: ? = B
```

Then:

```text
known: T, B
sum:   T + B + ? = 273
answer: ? = {
```

This continues until the whole flag is recovered.

---

## ✅ Final Solve Checklist

```text
[+] Identify maze.exe as PyInstaller
[+] Extract PyInstaller contents
[+] Recover ZIP password
[+] Recover PNG byte-offset seed
[+] AES-decrypt enc_maze.zip
[+] Generate random key with seed 493
[+] Deobfuscate every 10th byte
[+] Recover valid ELF binary
[+] Reverse adjacent-character sum checks
[+] Solve constraints with Python
[+] Recover final HTB flag
```

---

## 🧾 Clean Command Summary

```bash
# 1. Identify files
file maze.exe enc_maze.zip maze.png

# 2. Confirm PyInstaller
strings -a maze.exe | grep -Ei "pyi|pyinstaller|python|pyz"

# 3. Extract PyInstaller
python3 pyinstxtractor.py maze.exe

# 4. Search recovered strings/constants
strings -a maze.exe_extracted/* | grep -E "Y0u|maze|path|zip"

# 5. Run decryption/deobfuscation
python3 solve_stage1.py

# 6. Check recovered ELF
file dec_maze
chmod +x dec_maze

# 7. Reverse ELF
strings -a dec_maze
objdump -d -Mintel dec_maze | less

# 8. Solve final constraints
python3 solve_flag.py
```

---

## 🏆 Conclusion

This was a layered maze challenge. The solution required moving through multiple stages:

```text
PyInstaller reversing → AES ZIP extraction → PNG byte hint → binary deobfuscation → ELF reversing → constraint solving
```

The most important lesson is that every provided file mattered. The `.exe` gave the password and logic, the `.zip` contained the hidden binary, and the `.png` supplied the seed needed to repair that binary.

Once the ELF was restored, the final flag check was simple: a list of adjacent-character sums. Solving those sums produced the complete HTB flag.
