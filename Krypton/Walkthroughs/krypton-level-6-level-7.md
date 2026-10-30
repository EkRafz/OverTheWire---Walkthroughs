> **Level Goal:** The password for level 7 is in the file `krypton7`, encrypted with `encrypt6`. The keyfile is unreadable, but `encrypt6` will encrypt any plaintext you give it. The "random" number generator it uses is weak.

---

## Connection Details

| Field    | Value                                               |
| -------- | --------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                      |
| Port     | `2231`                                              |
| Username | `krypton6`                                          |
| Password | *[Krypton Level 5 → 6](krypton-level-5-level-6.md)* |

---

## Commands Used

- `ls -la` → List files with permissions and ownership
- `cat` → Print file contents
- `mktemp -d` → Create a temporary working directory
- `ln -s` → Symlink the keyfile into the working directory
- `chmod` → Make the directory writable for the setuid binary
- `python3 -c` → Generate a long run of `A`s and decrypt the result
- `encrypt6 <plaintext> <ciphertext>` → The level's encryption oracle (setuid `krypton7`)

> **Reference:** [Stream cipher | Wikipedia](https://en.wikipedia.org/wiki/Stream_cipher) · [One-time pad | Wikipedia](https://en.wikipedia.org/wiki/One-time_pad) · [Chosen-plaintext attack | Wikipedia](https://en.wikipedia.org/wiki/Chosen-plaintext_attack)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton6@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Survey the Level Directory

```bash
krypton6@krypton:~$ ls -la /krypton/krypton6
```

Output:

```
total 56
drwxr-xr-x 3 root     root      4096 Oct 25 13:38 ./
drwxr-xr-x 9 root     root      4096 Oct 25 13:38 ../
-rw-r----- 1 krypton6 krypton6   164 Oct 25 13:38 HINT1
-rw-r----- 1 krypton6 krypton6    11 Oct 25 13:38 HINT2
-rw-r----- 1 krypton6 krypton6  4342 Oct 25 13:38 README
-rwsr-x--- 1 krypton7 krypton6 16528 Oct 25 13:38 encrypt6*
-rw-r----- 1 krypton7 krypton7    11 Oct 25 13:38 keyfile.dat
-rw-r----- 1 krypton6 krypton6    15 Oct 25 13:38 krypton7
drwxr-xr-x 2 root     root      4096 Oct 25 13:38 onetime/
```

- `krypton7` → the ciphertext (15 bytes, so about 14 letters plus a newline)
- `encrypt6` → setuid `krypton7`, so it can read the key we can't
- `keyfile.dat` → owned by `krypton7`, unreadable to us. It is only 11 bytes, so the stored key is short
- `onetime/` → the one-time-pad demo files from the level page (`plain1`, `key1`, `cipher1`). Not needed for the password

---

### Step 3 — Read the Ciphertext

```bash
krypton6@krypton:~$ cat /krypton/krypton6/krypton7
PNUKLYLWRQKGKBE
```

---

### Understand the Level: One-Time Pad vs. Weak Keystream

> The level page opens with a **one-time pad** (OTP): a key as long as the plaintext, truly random, never reused. It is unbreakable, because every possible plaintext is equally consistent with a given ciphertext. `plain1`, `key1` and `cipher1` demonstrate it. The key is the first 256 bytes of _A Tale of Two Cities_, and without `key1` the ciphertext is useless.
> 
> The page then moves to **stream ciphers**: instead of a pre-shared key as long as the message, a generator produces a keystream on the fly, combined with the plaintext byte by byte (typically XOR). Security depends entirely on how unpredictable that keystream is. If it repeats, or can be predicted, the cipher collapses back to a repeating-key cipher like the Vigenère from [Krypton Level 5 → 6](krypton-level-5-level-6.md).
> 
> Here the page says directly that the generator is weak. Since we control the plaintext, we can make the keystream visible.

|Attack type|What you control|What you learn|
|---|---|---|
|Chosen-plaintext|Input to `encrypt6`|The keystream itself (if plaintext is known)|

The trick is the same as in [Krypton Level 2 → 3](krypton-level-2-level-3.md): encrypt a known plaintext and read the key out of the result. If plaintext is all `A` (shift 0 in Vigenère terms), the ciphertext letters _are_ the key letters.

---

### Step 4 — Set Up a Working Directory

Same setup as Level 2 → 3: `encrypt6` runs as `krypton7`, reads `keyfile.dat` from the current directory, and writes the output file you name there.

```bash
krypton6@krypton:~$ mktemp -d
/tmp/tmp.XXXXXXXXXX
krypton6@krypton:~$ cd /tmp/tmp.XXXXXXXXXX
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ ln -s /krypton/krypton6/keyfile.dat keyfile.dat
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ chmod 777 .
```

---

### Step 5 — Encrypt a Long Run of `A`s

Use a plaintext much longer than the ciphertext you need to decrypt, so any repetition in the keystream shows up:

```bash
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ python3 -c "print('A'*100)" > plaintext
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ /krypton/krypton6/encrypt6 plaintext ciphertext
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ cat ciphertext 
EICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZ
```

`encrypt6` takes two arguments, the plaintext file and the destination ciphertext file. Running it with one argument prints:

```
usage: encrypt6 foo bar
Where: foo is the file containing the plaintext and bar is the destination ciphertext file.
```

Look for a repeating block. The length of that block is the period of the keystream. Because the plaintext is `A` (index 0), each ciphertext letter equals the key letter at that position.

> **Verify before trusting the result:** run `encrypt6` a second time on the same plaintext and diff the outputs. If they differ, the keystream depends on the run (the "random" number), and a single capture is not enough, so the approach has to change. If they match, the keystream is fixed per position and the next step is valid.

```bash
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ /krypton/krypton6/encrypt6 plaintext run2
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ diff ciphertext run2 && echo identical
identical
```

---

### Step 6 — Extract the Keystream Period

```bash
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ python3 -c "
ct = open('ciphertext').read().strip()
for p in range(1, 60):
    if all(ct[i] == ct[i % p] for i in range(len(ct))):
        print('period', p, 'key', ct[:p])
        break
"

period 30 key EICTDGYIYZKTHNSIRFXYCPFUEOCKRN
```

---

### Step 7 — Decrypt the Password

Treat the recovered keystream as a repeating Vigenère key and subtract it from `krypton7`:

```bash
krypton6@krypton:/tmp/tmp.XXXXXXXXXX$ python3 -c "
key = 'EICTDGYIYZKTHNSIRFXYCPFUEOCKRN'
ct = 'PNUKLYLWRQKGKBE'
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

This is the password for [Krypton Level 7](krypton-level-7.md).

---

## Key Takeaways

- **A stream cipher is only as strong as its keystream.** A keystream that repeats turns the cipher into a repeating-key Vigenère, which is exactly what this series has shown is breakable. The OTP argument at the top of the level only holds if the key is random, as long as the message, and never reused.
- **A chosen-plaintext oracle exposes the key directly.** With an all-`A` plaintext, the ciphertext _is_ the keystream. No frequency analysis is needed, which is why the same trick worked in Level 2 → 3.
- **Weak PRNGs are a cryptographic failure.** The level page says the generator is weak. The practical consequence is that its output is predictable or periodic, and everything built on top of it inherits that weakness.

---