> **Level Goal:** Decode the Base64 string given on the level page to find the password for Level 1.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                           |
| Port     | `2231`                                                   |
| Username | `krypton1`                                               |
| Password | _encoded directly on the level info page (Base64)_       |

---

## Tools Used

- A web browser
- `base64` — encode/decode Base64 data from the command line

> **Reference:** [base64 man page](https://man7.org/linux/man-pages/man1/base64.1.html)

---

## Solution

### Step 1 — Read the Level Information Page

Krypton has no Level 0 SSH session of its own — the entry point is the level info page itself:

```
https://overthewire.org/wargames/krypton/krypton0.html
```

It states the password is encoded, and gives the encoded string directly:

```
S1JZUFRPTklTR1JFQVQ=
```

That string, plus the target host, port, and username (`krypton1`), is everything needed to reach Level 1.

---

### Understand Base64

> Base64 is an **encoding**, not an encryption scheme. It re-represents arbitrary binary data as printable ASCII text, using an alphabet of 64 characters (`A–Z`, `a–z`, `0–9`, `+`, `/`) plus `=` for padding at the end. There is no key and no secret involved, anyone holding the encoded string can recover the original data instantly. It is meant to make binary-safe data transportable through text-only channels (email, URLs, JSON), not to hide anything.

| Trait                              | What it means                                              |
| ---------------------------------- | ---------------------------------------------------------- |
| `=` or `==` at the end             | Padding, a strong visual signature that a string is Base64 |
| Alphabet restricted to A-Za-z0-9+/ | No spaces, no punctuation outside `+`, `/`, `=`            |
| Reversible with no key             | Decoding requires no secret, only the decoding algorithm   |

This is the easiest possible first step in the wargame: recognizing the pattern and running it through a decoder.

---

### Step 2 — Decode the String

```bash
$ echo "S1JZUFRPTklTR1JFQVQ=" | base64 -d
```

Output:

```
KRYPTONISGREAT
```

That is the password for this level.

---

### Step 3 — Connect to the Server

```bash
ssh krypton1@krypton.labs.overthewire.org -p 2231
```

Enter the decoded password (`KRYPTONISGREAT`) when prompted. A successful login drops into the `krypton1` home directory and prints the wargame's welcome banner, which notes that level files for the rest of the game live under `/krypton/`.
Now go to [Krypton Level 1 → Level 2](krypton-level-1-level-2.md)

---

## Key Takeaways

- **Base64 is not encryption.** It has no key, and decoding it requires no cryptanalysis, only recognizing the alphabet and padding pattern, then running it through any standard decoder. Treating a Base64 string as if it were secured is a common and serious misconception.
- **Padding and character set are a reliable fingerprint.** A string built only from `A-Za-z0-9+/` and ending in `=` or `==` is almost always Base64, which makes it trivial to spot in source code, config files, or HTTP traffic.

---