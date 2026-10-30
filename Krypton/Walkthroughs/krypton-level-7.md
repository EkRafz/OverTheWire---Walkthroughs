> **Level Goal:** This is the last level. There is no password for a next level.

---

## Connection Details

| Field    | Value                                                              |
| -------- | ------------------------------------------------------------------ |
| Host     | `krypton.labs.overthewire.org`                                     |
| Port     | `2231`                                                             |
| Username | `krypton7`                                                         |
| Password | *found in [Krypton Level 6 → Level 7](krypton-level-6-level-7.md)* |

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton7@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Read the README

The home directory is empty. As in every previous level, the level files are under `/krypton/`:

```bash
krypton7@krypton:~$ ls
krypton7@krypton:~$ cd /krypton/krypton7/
krypton7@krypton:/krypton/krypton7$ ls
README
krypton7@krypton:/krypton/krypton7$ cat README
Congratulations on beating Krypton!
```

That's it. You've completed Krypton.

---

## What You've Covered

Across 7 levels, Krypton introduced the core of classical cryptography and cryptanalysis:

|Category|Skills|
|---|---|
|**Encoding**|Base64 (an encoding, not encryption), `base64 -d`|
|**Classical ciphers**|ROT13, Caesar, monoalphabetic substitution, Vigenère|
|**Cryptanalysis**|Frequency analysis, chi-squared scoring, Index of Coincidence, column splitting|
|**Attack models**|Chosen-plaintext attack against an encryption oracle|
|**Stream ciphers**|One-time pad, keystreams, why a weak or repeating keystream breaks the cipher|
|**Tools**|`tr`, `fold`, `sort`, `mktemp`, `ln -s`, `chmod`, setuid binaries, Python scripts|

---