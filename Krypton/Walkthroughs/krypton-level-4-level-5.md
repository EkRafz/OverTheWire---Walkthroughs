> **Level Goal:** The password for level 5 is in the file `krypton5`, encrypted with a Vigenère cipher. You have two longer English-language ciphertexts (`found1`, `found2`) encrypted with the same key. You know the key length: **6**.

> **NOTE:** While doing this level I had the help of an AI

---

## Connection Details

| Field    | Value                                               |
| -------- | --------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                      |
| Port     | `2231`                                              |
| Username | `krypton4`                                          |
| Password | *[Krypton Level 3 → 4](krypton-level-3-level-4.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `cat` — Print file contents
- `tr -d` — Strip whitespace/newlines from input
- `wc -c` — Count characters
- A short Python script — split ciphertext into per-key-position columns and run chi-squared frequency analysis on each

> **Reference:** [Vigenère cipher — Wikipedia](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton4@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Survey the Level Directory

```bash
krypton4@krypton:~$ ls -la /krypton/krypton4
```

Output:

```
drwxr-xr-x 2 root     root     4096 Jun 24 15:00 ./
drwxr-xr-x 9 root     root     4096 Jun 24 15:00 ../
-rw-r----- 1 krypton4 krypton4  287 Jun 24 15:00 HINT
-rw-r----- 1 krypton4 krypton4 1385 Jun 24 15:00 README
-rw-r----- 1 krypton4 krypton4 1740 Jun 24 15:00 found1
-rw-r----- 1 krypton4 krypton4 2943 Jun 24 15:00 found2
-rw-r----- 1 krypton4 krypton4   10 Jun 24 15:00 krypton5
```

No `encrypt` binary this time, and no keyfile. The attack surface is the two known-plaintext-language files `found1` and `found2`, plus the fact that the key length is given: 6.

---

### Understand the Vigenère Cipher

> A **Vigenère cipher** is a **polyalphabetic** substitution cipher: instead of one fixed shift for every letter (as in a Caesar cipher), the shift changes depending on position, cycling through a repeating key. If the key is `GOLD` (shifts 6, 14, 11, 3), the 1st, 5th, 9th... plaintext letters are all shifted by the 1st key letter's value, the 2nd, 6th, 10th... by the 2nd key letter's value, and so on.
> 
> The critical consequence: **a single plaintext letter no longer maps to a single ciphertext letter.** `E` might become `K` in one position and `S` in another, depending on which key letter lines up with it. This defeats naive single-alphabet frequency analysis on the raw ciphertext.

### Why Knowing the Key Length Breaks This Open

> If the key length is unknown, the first job is normally the **Kasiski examination** or an **index of coincidence** scan to find it. Here, the level tells you directly: the key length is **6**. That single fact reduces the entire problem back to something already solved in Level 2→3 and 3→4.
> 
> If you split a ciphertext into 6 interleaved streams, every 6th character starting at position 0 goes into stream 1, every 6th starting at position 1 goes into stream 2, and so on, each individual stream was encrypted with **one single key letter** the whole way through. That means each of the 6 streams is just a **Caesar cipher**, and each can be broken independently with frequency analysis.

|Concept|Role here|
|---|---|
|Key length = 6|Given by the level|
|Column splitting|Divides ciphertext into 6 monoalphabetic (Caesar) streams|
|Per-column frequency analysis|Each column solved independently, like Level 2→3 / 3→4|
|Recombination|The 6 recovered shifts, in order, spell the key|

---

### Step 3 — The File-Alignment Pitfall (Read This Before Splitting)

`found1` and `found2` are **two separate ciphertexts**, each independently encrypted starting the key back at position 0. Their raw lengths (after stripping the 5-letter-block spacing) are:

```bash
krypton4@krypton:/krypton/krypton4$ mktemp -d # QUESTO E' IMPORTANTE
/tmp/tmp.xxxxxxxxxx

krypton4@krypton:/tmp/tmp.xxxxxxxxxx$ tr -d '[:space:]' < /krypton/krypton4found1 > f1.txt

krypton4@krypton:/tmp/tmp.xxxxxxxxxx$ tr -d '[:space:]' < /krypton/krypton4found2 > f2.txt

krypton4@krypton:/tmp/tmp.xxxxxxxxxx$ wc -c f1.txt f2.txt
```

Output:

```
1450 f1.txt
2453 f2.txt
```

**This is the trap.** 1450 mod 6 = 4, and 2453 mod 6 = 5, neither file's length is a multiple of 6. If you naively concatenate `f1.txt` and `f2.txt` into one string and then split _that_ by position mod 6, every character of `found2` lands in the wrong column: `found2`'s own position-0 character (which really was encrypted with the key's first letter) ends up 4 slots off in the combined stream, because `found1` didn't end on a column boundary.

The fix: **split each file into its own 6 columns first, independently**, then merge column 0 of `found1` with column 0 of `found2`, column 1 with column 1, and so on. Never concatenate the raw ciphertexts before splitting.

```
krypton4@krypton:/tmp/tmp.xxxxxxxxxx$ nano decipher.py
```

```python
def split_cols(text, n=6):
    cols = [''] * n
    for i, ch in enumerate(text):
        cols[i % n] += ch
    return cols

f1 = open('f1.txt').read().strip()
f2 = open('f2.txt').read().strip()

cols1 = split_cols(f1)
cols2 = split_cols(f2)
combined_cols = [cols1[i] + cols2[i] for i in range(6)]
```

Each `combined_cols[i]` is now a clean, single-key-letter Caesar stream, roughly 650 characters long, pooled from both source files.

---

### Step 4 — Frequency-Analyze Each Column Independently

For each column, test all 26 possible shifts and score them against standard English letter frequencies using a chi-squared statistic (a more robust version of the "top letter = E" trick used in Level 3→4 — it weighs the whole distribution, not just the single most common letter):

```python
freq = {
'A':8.167,'B':1.492,'C':2.782,'D':4.253,'E':12.702,'F':2.228,'G':2.015,
'H':6.094,'I':6.966,'J':0.153,'K':0.772,'L':4.025,'M':2.406,'N':6.749,
'O':7.507,'P':1.929,'Q':0.095,'R':5.987,'S':6.327,'T':9.056,'U':2.758,
'V':0.978,'W':2.360,'X':0.150,'Y':1.974,'Z':0.074
}

def chi_squared(text, shift):
    n = len(text)
    counts = [0]*26
    for ch in text:
        idx = (ord(ch) - 65 - shift) % 26
        counts[idx] += 1
    chi = 0
    for i in range(26):
        letter = chr(65+i)
        expected = freq[letter]/100 * n
        chi += (counts[i]-expected)**2/expected
    return chi

for col, text in enumerate(combined_cols):
    scores = sorted((chi_squared(text, s), s) for s in range(26))
    print(col, scores[:2])
```

Result, best shift per column, with the runner-up for comparison:

|Column|Best shift|Chi² (best)|Chi² (2nd best)|Key letter|
|---|---|---|---|---|
|0|5|29.7|2005.9|`F`|
|1|17|52.1|2112.8|`R`|
|2|4|39.9|2205.4|`E`|
|3|10|41.1|2391.4|`K`|
|4|4|18.1|2450.7|`E`|
|5|24|40.5|1760.7|`Y`|

The gap between best and second-best is enormous in every column (tens vs. thousands), this is not a close call, and confirms the per-file column splitting fixed the alignment problem from Step 3. Splitting naively (concatenate-then-split) was tried first and produced a much weaker, ambiguous fit, that mismatch is the signature of the alignment bug.

---

### Step 5 — Recover the Key

Reading the key letters in column order (0 through 5):

```
F R E K E Y
```

**Key: `FREKEY`**

---

### Step 6 — Verify Against the Known Plaintexts

Decrypting `found1` and `found2` with `FREKEY` (standard Vigenère decryption — subtract each key letter's shift, cycling the key every 6 characters, restarting the key at position 0 for each file separately):ììììì

```bash
krypton4@krypton:/krypton/krypton4$ python3 -c "
key = 'FREKEY'
def dec(text):
    out = []
    for i, ch in enumerate(text):
        s = ord(key[i % 6]) - 65
        out.append(chr((ord(ch) - 65 - s) % 26 + 65))
    return ''.join(out)
print(dec(open('f1.txt').read().strip())[:120])
"
```

Output (start of `found1`):

```
THESOLDIERWITHTHEGREENWHISKERSLEDTHEMTHROUGHTHESTREETSOFTHEEMERALDCITYUNTILTHEYREACHEDTHEROOMWHERETHEGUARDIANOFTHEGATESL
```

Clean grammatical English confirms the key.

---

### Step 7 — Decrypt the Password

```bash
krypton4@krypton:/krypton/krypton4$ cat krypton5
```

Output:

```
HCIKV RJOX
```

Decrypt with the same key (restarting at position 0, since `krypton5` is its own file):

```bash
krypton4@krypton:/krypton/krypton4$ python3 -c "
key = 'FREKEY'
ct = 'HCIKVRJOX'
out = []
for i, ch in enumerate(ct):
    s = ord(key[i % 6]) - 65
    out.append(chr((ord(ch) - 65 - s) % 26 + 65))
print(''.join(out))
"
```

Output:

```
<password>
```

This password  for [Krypton Level 5 → 6](krypton-level-5-level-6.md).

---

## Key Takeaways

- **Knowing the key length turns a Vigenère cipher into N independent Caesar ciphers.** Splitting ciphertext into N interleaved streams (one per key position) reduces a polyalphabetic problem to N monoalphabetic problems, each solvable with the same frequency-analysis technique used against simple substitution and Caesar ciphers.

---