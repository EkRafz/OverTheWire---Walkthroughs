> **Level Goal:** The password for level 6 is in the file `krypton6`, encrypted with a Vigenère cipher. You have two longer English-language ciphertexts (`found1`, `found2`) encrypted with the same key. **The key length is not given** — you must find it yourself.

---

## Connection Details

| Field    | Value                                               |
| -------- | --------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                      |
| Port     | `2231`                                              |
| Username | `krypton5`                                          |
| Password | *[Krypton Level 4 → 5](krypton-level-4-level-5.md)* |

---

## Commands Used

- `ls -la` → List all files including hidden, with permissions and ownership
- `cat` → Print file contents
- `tr -d` → Strip whitespace/newlines from input
- `wc -c` → Count characters
- A Python script → Index of Coincidence scan to find key length, then per-column chi-squared frequency analysis to recover each key letter

> **Reference:** [Vigenère cipher | Wikipedia](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) · [Index of Coincidence | Wikipedia](https://en.wikipedia.org/wiki/Index_of_coincidence)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton5@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Survey the Level Directory

```bash
krypton5@krypton:~$ ls -la /krypton/krypton5
```

Output:

```
total 28
drwxr-xr-x 2 root     root     4096 Jun 24 15:00 .
drwxr-xr-x 9 root     root     4096 Jun 24 15:00 ..
-rw-r----- 1 krypton5 krypton5  151 Jun 24 15:00 README
-rw-r----- 1 krypton5 krypton5 1776 Jun 24 15:00 found1
-rw-r----- 1 krypton5 krypton5 1915 Jun 24 15:00 found2
-rw-r----- 1 krypton5 krypton5 2110 Jun 24 15:00 found3
-rw-r----- 1 krypton5 krypton5    7 Jun 24 15:00 krypton6
```

Three ciphertext files this time (`found1`, `found2`, `found3`), no HINT file, finding the key length is entirely on you.

---

### Understand the Index of Coincidence

> The **Index of Coincidence** (IoC) measures how "uneven" a frequency distribution is, specifically, the probability that two randomly chosen letters from the text are the same. Natural English has IoC ≈ **0.065** because a few letters (`E`, `T`, `A`…) dominate. A perfectly uniform distribution (all 26 letters equally likely) has IoC ≈ **0.038**.
> 
> A Vigenère cipher with key length _n_ splits plaintext into _n_ independent Caesar streams. Each stream is perfectly uneven (IoC ≈ 0.065), but when you view the _whole_ ciphertext as one stream, the shifts cancel each other out and the distribution looks flatter (IoC closer to 0.038). The longer the key, the flatter the whole-text IoC, but more importantly: **if you re-split the ciphertext into _n_ columns and n happens to be the real key length, every column will have IoC ≈ 0.065 again.** If n is wrong, columns still look flat.
> 
> The attack: scan candidate key lengths 1 through 20, compute the average per-column IoC for each, and look for the candidate whose average IoC spikes up to ≈ 0.065. That spike reveals the true key length, without needing any known plaintext.

|Average per-column IoC|Interpretation|
|---|---|
|≈ 0.065|Columns line up with one key letter each → correct n|
|≈ 0.038 – 0.050|Columns still mixed → wrong n|

---

### Step 3 — Strip Whitespace and Combine

As in [Krypton Level 4 → 5](krypton-level-4-level-5.md), each file must be split into its own per-key-position columns separately, then merged column by column. First, strip the 5-letter-block spacing:

```bash
krypton5@krypton:~$ mktemp -d
/tmp/tmp.XyZaBcDe
krypton5@krypton:~$ cd /tmp/tmp.XyZaBcDe

krypton5@krypton:/tmp/tmp.XyZaBcDe$ tr -d '[:space:]' < /krypton/krypton5/found1 > f1.txt
krypton5@krypton:/tmp/tmp.XyZaBcDe$ tr -d '[:space:]' < /krypton/krypton5/found2 > f2.txt
krypton5@krypton:/tmp/tmp.XyZaBcDe$ tr -d '[:space:]' < /krypton/krypton5/found3 > f3.txt

krypton5@krypton:/tmp/tmp.XyZaBcDe$ wc -c f1.txt f2.txt f3.txt
```

Output:

```
1480 f1.txt
1596 f2.txt
1759 f3.txt
4835 total
```

---

### Step 4 — Find the Key Length with the Index of Coincidence

```bash
krypton5@krypton:/tmp/tmp.XyZaBcDe$ nano ioc.py
```

```python
def ioc(text):
    n = len(text)
    if n < 2:
        return 0
    counts = [0] * 26
    for ch in text:
        counts[ord(ch) - 65] += 1
    return sum(c * (c - 1) for c in counts) / (n * (n - 1))

def split_cols(text, key_len):
    cols = [''] * key_len
    for i, ch in enumerate(text):
        cols[i % key_len] += ch
    return cols

f1 = open('f1.txt').read().strip()
f2 = open('f2.txt').read().strip()
f3 = open('f3.txt').read().strip()

print(f"{'Key len':>8}  {'Avg IoC':>8}")
print("-" * 20)
for n in range(1, 21):
    cols1 = split_cols(f1, n)
    cols2 = split_cols(f2, n)
    cols3 = split_cols(f3, n)
    combined = [cols1[i] + cols2[i] + cols3[i] for i in range(n)]
    avg = sum(ioc(c) for c in combined) / n
    print(f"{n:>8}  {avg:>8.4f}")
```

```bash
krypton5@krypton:/tmp/tmp.XyZaBcDe$ python3 ioc.py
```

Output:

```
 Key len   Avg IoC
--------------------
       1    0.0410
       2    0.0410
       3    0.0495
       4    0.0409
       5    0.0409
       6    0.0496
       7    0.0409
       8    0.0408
       9    0.0654
      10    0.0410
      11    0.0409
      12    0.0496
      13    0.0409
      14    0.0409
      15    0.0494
      16    0.0407
      17    0.0411
      18    0.0656
      19    0.0414
      20    0.0406
```

Key length **9** spikes to ≈ 0.064, cleanly above the noise floor of ≈ 0.044. Length 18 also spikes (it is a multiple of 9, so 9-wide columns split again evenly), which confirms the finding. **The key length is 9.**

---

### Step 5 — Recover Each Key Letter with Chi-Squared

Same technique as [Krypton Level 4 → 5](krypton-level-4-level-5.md): split each file into 9 columns independently, merge column-by-column, then test all 26 shifts against the English frequency table and pick the one with the lowest chi-squared score:

```bash
krypton5@krypton:/tmp/tmp.XyZaBcDe$ nano crack.py
```

```python
freq = {
    'A':8.167,'B':1.492,'C':2.782,'D':4.253,'E':12.702,'F':2.228,'G':2.015,
    'H':6.094,'I':6.966,'J':0.153,'K':0.772,'L':4.025,'M':2.406,'N':6.749,
    'O':7.507,'P':1.929,'Q':0.095,'R':5.987,'S':6.327,'T':9.056,'U':2.758,
    'V':0.978,'W':2.360,'X':0.150,'Y':1.974,'Z':0.074
}

def chi_squared(text, shift):
    n = len(text)
    counts = [0] * 26
    for ch in text:
        counts[(ord(ch) - 65 - shift) % 26] += 1
    return sum((counts[i] - freq[chr(65+i)]/100 * n)**2 / (freq[chr(65+i)]/100 * n)
               for i in range(26))

def split_cols(text, key_len):
    cols = [''] * key_len
    for i, ch in enumerate(text):
        cols[i % key_len] += ch
    return cols

KEY_LEN = 9   # update this if your IoC scan gives a different value
f1 = open('f1.txt').read().strip()
f2 = open('f2.txt').read().strip()
f3 = open('f3.txt').read().strip()

cols1 = split_cols(f1, KEY_LEN)
cols2 = split_cols(f2, KEY_LEN)
cols3 = split_cols(f3, KEY_LEN)
combined = [cols1[i] + cols2[i] + cols3[i] for i in range(KEY_LEN)]

key = ''
for col_idx, col in enumerate(combined):
    best = min(range(26), key=lambda s: chi_squared(col, s))
    key += chr(65 + best)
    scores = sorted((chi_squared(col, s), s) for s in range(26))
    print(f"Col {col_idx}: shift={best} letter={chr(65+best)}  "
          f"chi²={scores[0][0]:.1f}  runner-up={scores[1][0]:.1f}")

print(f"\nKey: {key}")
```

```bash
krypton5@krypton:/tmp/tmp.XyZaBcDe$ python3 crack.py
```

Output:

```
Col 0: shift=10 letter=K  chi²=40.7  runner-up=1692.9
Col 1: shift=4 letter=E  chi²=21.9  runner-up=2167.9
Col 2: shift=24 letter=Y  chi²=22.8  runner-up=2091.3
Col 3: shift=11 letter=L  chi²=21.5  runner-up=1690.3
Col 4: shift=4 letter=E  chi²=25.7  runner-up=1873.6
Col 5: shift=13 letter=N  chi²=25.0  runner-up=2267.4
Col 6: shift=6 letter=G  chi²=12.7  runner-up=1920.8
Col 7: shift=19 letter=T  chi²=30.1  runner-up=1346.8
Col 8: shift=7 letter=H  chi²=32.6  runner-up=1791.4

Key: KEYLENGTH
```

---

### Step 6 — Verify Against a Known Ciphertext

Decrypting the first 120 characters of `found1` with `KEYLENGTH` (restarting key at position 0 for each file):

```bash
krypton5@krypton:/tmp/tmp.XyZaBcDe$ python3 -c "
key = 'KEYLENGTH'
def dec(text):
    out = []
    for i, ch in enumerate(text):
        s = ord(key[i % len(key)]) - 65
        out.append(chr((ord(ch) - 65 - s) % 26 + 65))
    return ''.join(out)
print(dec(open('f1.txt').read().strip())[:120])
"
```

Output (start of `found1`):

```
ITWASTHEBESTOFTIMESITWASTHEWORSTOFTIMESITWASTHEAGEOFWISDOMITWASTHEAGEOFFOOLISHNESSITWASTHEEPOCHOFBELIEFITWASTHEEPOCHOFIN
```

Clean grammatical English, the key is confirmed.

---

### Step 7 — Decrypt the Password

```bash
krypton5@krypton:/krypton/krypton5$ cat krypton6
```

Output:

```
BELOS Z
```

Decrypt with `KEYWORDSA` (key restarts at position 0, since `krypton6` is its own file):

```bash
krypton5@krypton:/tmp/tmp.XyZaBcDe$ python3 -c "
key = 'KEYLENGTH'
ct = 'BELOSZ'
out = []
for i, ch in enumerate(ct):
    s = ord(key[i % len(key)]) - 65
    out.append(chr((ord(ch) - 65 - s) % 26 + 65))
print(''.join(out))
"
```

Output:

```
<password>
```

This is the password for [Krypton Level 6 → 7](krypton-level-6-level-7.md).

---

## Key Takeaways

- **The Index of Coincidence finds the key length without any known plaintext.** IoC measures how "English-like" a distribution is. Splitting ciphertext into columns at the correct key length makes every column monoalphabetic and gives average IoC ≈ 0.065, identical to natural English. Wrong key lengths leave columns still scrambled (IoC ≈ 0.038–0.050). The correct length stands out as a clear spike.
- **Multiples of the true key length also spike.** Length 18 peaks alongside 9 because every 9th-position column is still a single Caesar stream when split at 18. Always take the smallest spiking value as the key length.
- **Once key length is known, the problem collapses back to Level 3→4.** IoC finds _n_; then chi-squared frequency analysis on each of the _n_ columns, exactly as in the previous two levels, recovers every key letter independently. The two attacks chain: IoC first, then per-column frequency analysis.