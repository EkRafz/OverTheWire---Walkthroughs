> **Level Goal:** The password for level 4 is in the file `krypton4`, encrypted with an unknown monoalphabetic substitution cipher. You have no encryption mechanism this time, but you have three additional ciphertext files (`found1`, `found2`, `found3`) encrypted with the same key. The plaintexts are in American English.

---

## Connection Details

| Field    | Value                                               |
| -------- | --------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                      |
| Port     | `2231`                                              |
| Username | `krypton3`                                          |
| Password | *[Krypton Level 2 → 3](krypton-level-2-level-3.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `cat` — Print file contents
- `tr` — Translate characters
- `for` loop — Iterate over values in Bash
- `wc -c` — Count characters
- `sort` — Sort input lines
- `fold -w1` — Split input into one character per line

> **Reference:**  [tr man page](https://man7.org/linux/man-pages/man1/tr.1.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton3@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Survey the Level Directory

```bash
krypton3@krypton:~$ ls -la /krypton/krypton3
```

Output:

```
drwxr-xr-x 2 root     root     4096 Apr  1 04:20 .
drwxr-xr-x 8 root     root     4096 Apr  1 04:20 ..
-rw-r----- 1 krypton3 krypton3  785 Apr  1 04:20 HINT1
-rw-r----- 1 krypton3 krypton3 2594 Apr  1 04:20 HINT2
-rw-r----- 1 krypton3 krypton3 1285 Apr  1 04:20 README
-rw-r----- 1 krypton3 krypton3 5391 Apr  1 04:20 found1
-rw-r----- 1 krypton3 krypton3 3604 Apr  1 04:20 found2
-rw-r----- 1 krypton3 krypton3 2534 Apr  1 04:20 found3
-rw-r----- 1 krypton3 krypton3   43 Apr  1 04:20 krypton4
```

No `encrypt` binary this time. The attack surface is `found1`, `found2`, `found3`.

---

### Step 3 — Count Letter Frequencies Across All Found Files

Combine all three ciphertext files and count how many times each letter appears, sorted from most to least frequent:

```bash
krypton3@krypton:/krypton/krypton3$ for i in {A..Z}; do
  cnt=$(cat found1 found2 found3 | tr -cd "$i" | wc -c)
  printf "%d %s\n" "$cnt" "$i"
done | sort -nr
```

Output (abridged):

```
456 S
340 Q
301 J
257 U
246 B
240 N
...
4 R
4 H
2 P
```

`S` dominates. In English, `E` dominates. The mapping `S → E` is the first anchor.

Sorted ciphertext frequency order: `S Q J U B N G C D Z V W M Y T X K E L A F I O R H P`

Standard English frequency order: `E T A S O I N H R D L C U M W F G Y P B V K J X Q Z`

The initial frequency-matched substitution table (ciphertext → plaintext):

```
S→E  Q→T  J→A  U→S  B→O  N→I  G→N  C→H  D→R  Z→C
V→L  W→D  M→U  Y→P  T→Y  X→F  K→W  E→G  L→M  A→B
F→K  I→V  O→X  R→Q  H→J  P→Z
```

---

### Step 4 — Refine with Common Words

Raw frequency matching is an approximation. The refinement comes from pattern recognition: look for common short words (`THE`, `AND`, `OF`, `TO`, `IS`) in partially decoded text.

The most reliable anchor is `THE`, the most common three-letter English word. Count trigraphs (3-letter sequences) in the combined ciphertext:

```bash
krypton3@krypton:/krypton/krypton3$ cat found1 found2 found3 | tr -d ' \n' | \
  fold -w3 | sort | uniq -c | sort -nr | head -10
```

The trigraph `JDS` appears most frequently. Mapping `JDS → THE` gives `J→T`, `D→H`, `S→E` — consistent with the frequency analysis. This locks three letters.

Working through the partially decoded text iteratively, substituting known mappings and reading for English words, converges on the complete substitution table:

```
Ciphertext: S Q J U B N G C D Z V W M Y T X K E L A F I O R H P
Plaintext:  E A T S O R N I H C L D U P Y F W G M B K V X Q J Z
```

---

### Step 5 — Decrypt the Password

```bash
krypton3@krypton:/krypton/krypton3$ cat krypton4 | tr 'SQJUBNGCDZVWMYTXKELAFIORHP' 'EATSORNIHCLDUPYFWGMBKVXQJZ'
```

Output:

```
WELLD ONETH ELEVE LFOUR PASSW ORDIS <password>
```

This is the password for [Krypton Level 4 → 5](krypton-level-4-level-5.md).

---

## Key Takeaways

- **A monoalphabetic substitution cipher is broken by frequency analysis, not brute force.** With 26! possible keys, exhaustive search is hopeless. But because natural language has predictable letter frequencies, even a few hundred characters of ciphertext is enough to recover the key. More ciphertext means more reliable counts, which is why the level provides three intercepted files.
- **Trigraph `JDS → THE` is the strongest single anchor.** `THE` is the most common English trigraph and appears so frequently that its ciphertext equivalent usually tops the trigraph count. Identifying it pins three letters at once, which cascades through the rest of the mapping.

---