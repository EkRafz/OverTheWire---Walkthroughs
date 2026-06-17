> **Level Goal:** The password for level 2 is in the file `krypton2`. It is encrypted using a simple rotation (ROT13), kept in non-standard ciphertext format — plaintext word boundaries are preserved instead of being grouped into 5-letter clusters.

---

## Connection Details

| Field    | Value                                               |
| -------- | --------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                      |
| Port     | `2231`                                              |
| Username | `krypton1`                                          |
| Password | *[Krypton Level 0 → 1](krypton-level-0-level-1.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `cat` — Print file contents
- `tr` — Translate or delete characters from standard input

> **Reference:** [tr man page](https://man7.org/linux/man-pages/man1/tr.1.html) · [Caesar cipher — Wikipedia](https://en.wikipedia.org/wiki/Caesar_cipher)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton1@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Read the README

Krypton stores its level files under `/krypton/`, with a subdirectory per level:

```bash
krypton1@krypton:~$ cd /krypton/krypton1
krypton1@krypton:/krypton/krypton1$ ls
README  krypton2
krypton1@krypton:/krypton/krypton1$ cat README
```

Output:

```
Welcome to Krypton!

This game is intended to give hands on experience with cryptography
and cryptanalysis.  The levels progress from classic ciphers, to modern,
easy to harder.

Although there are excellent public tools, like cryptool,to perform
the simple analysis, we strongly encourage you to try and do these
without them for now.  We will use them in later excercises.

** Please try these levels without cryptool first **


The first level is easy.  The password for level 2 is in the file 
'krypton2'.  It is 'encrypted' using a simple rotation called ROT13.  
It is also in non-standard ciphertext format.  When using alpha characters for
cipher text it is normal to group the letters into 5 letter clusters, 
regardless of word boundaries.  This helps obfuscate any patterns.

This file has kept the plain text word boundaries and carried them to
the cipher text.

Enjoy!
```

The README confirms the cipher (ROT13) and flags the one deviation from standard practice: word boundaries (spaces) were left intact instead of being stripped and re-grouped into 5-letter blocks. That's a free hint, the message will read as recognizable words once decoded, not an unbroken block of letters.

> This level is pretty similar to  this level [bandit-level-11-level-12](Bandit/Walkthroughs/bandit-level-11-level-12).

---

### Step 3 — Read the Ciphertext

```bash
krypton1@krypton:/krypton/krypton1$ cat krypton2
```

Output:

```
YRIRY GJB CNFFJBEQ EBGGRA
```

---

### Understand ROT13

> ROT13 ("rotate by 13") is a special case of the **Caesar cipher**, a substitution cipher where every letter is shifted a fixed number of places through the alphabet. With a 26-letter alphabet, a shift of 13 is exactly half the alphabet's length, which gives ROT13 a unique property: applying it twice returns the original text. Encryption and decryption are the same operation. It provides no real security, it's historically used to obscure spoilers or punchlines from a casual glance, not to protect secrets, which makes it the natural "introductory" cipher for a cryptography wargame.

|Plaintext letter|Shift|Ciphertext letter|
|---|---|---|
|`A`|+13|`N`|
|`L`|+13|`Y`|
|`Z`|+13|`M`|

---

### Step 4 — Decode with `tr`

`tr` translates characters from one set to another, position by position. Mapping `A-Za-z` onto an alphabet rotated by 13 positions (`N-ZA-Mn-za-m`) performs ROT13 on whatever text is piped into it:

```bash
krypton1@krypton:/krypton/krypton1$ cat krypton2 | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Output:

```
<password>
```

The decoded line states the password directly: it is the password for [Krypton Level 2 → 3](krypton-level-2-level-3.md).

---

## Key Takeaways

- **ROT13 has no key.** Unlike a general Caesar cipher, where the shift value (1–25) must be found or guessed, ROT13's shift is fixed and well-known.
- **`tr` is a one-line cipher engine for fixed substitutions.** Mapping `A-Za-z` to a rotated alphabet handles the entire alphabet, upper and lower case, in a single command, no scripting needed for ROT13 or any other fixed substitution shift.

---