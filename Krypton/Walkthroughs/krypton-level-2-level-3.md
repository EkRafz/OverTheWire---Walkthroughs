> **Level Goal:** The password for level 3 is in the file `krypton3`. It is encrypted with a Caesar Cipher. Figure out the shift and decrypt the ciphertext.

---

## Connection Details

| Field    | Value                                               |
| -------- | --------------------------------------------------- |
| Host     | `krypton.labs.overthewire.org`                      |
| Port     | `2231`                                              |
| Username | `krypton2`                                          |
| Password | *[Krypton Level 1 → 2](krypton-level-1-level-2.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `cat` — Print file contents
- `mktemp -d` — Create a temporary directory with a unique name
- `ln -s` — Create a symbolic link
- `chmod` — Change file permissions
- `echo` — Write a string to stdout
- `tr` — Translate characters

> **Reference:** [Caesar cipher — Wikipedia](https://en.wikipedia.org/wiki/Caesar_cipher) · [tr man page](https://man7.org/linux/man-pages/man1/tr.1.html) · [mktemp man page](https://man7.org/linux/man-pages/man1/mktemp.1.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh krypton2@krypton.labs.overthewire.org -p 2231
```

---

### Step 2 — Survey the Level Directory

```bash
krypton2@krypton:~$ ll /krypton/krypton2
```

Output:

```
drwxr-xr-x 2 root     root      4096 Jun 24 15:00 ./
drwxr-xr-x 9 root     root      4096 Jun 24 15:00 ../
-rw-r----- 1 krypton2 krypton2  1815 Jun 24 15:00 README
-rwsr-x--- 1 krypton3 krypton2 16336 Jun 24 15:00 encrypt*
-rw-r----- 1 krypton3 krypton3    27 Jun 24 15:00 keyfile.dat
-rw-r----- 1 krypton2 krypton2    13 Jun 24 15:00 krypton3
```

Three things stand out:

- `krypton3` — the ciphertext file containing the password
- `keyfile.dat` — the actual key, owned by `krypton3` and unreadable directly
- `encrypt` — a setuid binary (owned by `krypton3`) that uses the key to encrypt anything you feed it

The key is hidden, but the encryption oracle (`encrypt`) is available. That's enough.

---

### Step 3 — Read the Ciphertext

```bash
krypton2@krypton:~$ cat /krypton/krypton2/krypton3
```

Output:

```
OMQEMDUEQMEK
```

The ciphertext is 12 characters, in 5-letter group format (non-standard here, word boundaries were preserved).

---

### Understand the Caesar Cipher and the Chosen-Plaintext Attack

> A **Caesar cipher** shifts every letter in the plaintext a fixed number of positions through the alphabet. With a shift of 12, `A` becomes `M`, `B` becomes `N`, and so on, wrapping around at `Z`. The cipher has only 25 possible keys (shifts 1–25), making brute force trivial. But there is an even cleaner path here: a **chosen-plaintext attack**.
> 
> Since `encrypt` uses the same key as the ciphertext, you can feed it a plaintext you fully control, the alphabet in order, and observe what comes out. The output is the complete substitution table: position 0 in the output is what `A` encrypts to, position 1 is what `B` encrypts to, and so on. Once you have the table, decrypting the password is a single `tr` call.

|Attack type|What you control|What you learn|
|---|---|---|
|Brute force|Nothing|Try all 25 shifts|
|Chosen-plaintext|Input to `encrypt`|Exact substitution table|

One encryption of the alphabet is enough to read the key completely. The level description says "one shot can solve it", this is that shot.

---

### Step 4 — Set Up a Working Directory

The `encrypt` binary looks file containing the plaintext in the **current working directory**. `/krypton/krypton2/` is read-only, so you cannot run `encrypt` from there. The solution is to create a writable directory in `/tmp`, symlink the keyfile into it, and run `encrypt` from there.

```bash
krypton2@krypton:~$ mktemp -d
/tmp/tmp.aBcDeFgH
krypton2@krypton:~$ cd /tmp/tmp.aBcDeFgH
krypton2@krypton:/tmp/tmp.aBcDeFgH$ ln -s /krypton/krypton2/keyfile.dat keyfile.dat
krypton2@krypton:/tmp/tmp.aBcDeFgH$ chmod 777 .
```

`chmod 777 .` is required because `encrypt` runs as `krypton3` (setuid), and `krypton3` must be able to write the output `ciphertext` file into the directory.

---

### Step 5 — Encrypt a Known Plaintext

Create a file containing the full alphabet in order and encrypt it:

```bash
krypton2@krypton:/tmp/tmp.aBcDeFgH$ echo "ABCDEFGHIJKLMNOPQRSTUVWXYZ" > plaintext
krypton2@krypton:/tmp/tmp.aBcDeFgH$ /krypton/krypton2/encrypt plaintext
krypton2@krypton:/tmp/tmp.aBcDeFgH$ cat ciphertext
```

Output:

```
MNOPQRSTUVWXYZABCDEFGHIJKL
```

Reading the output: `A` encrypts to `M` (the 13th letter, 0-indexed position 12). The shift is **+12**.

|Position|Plaintext|Ciphertext|
|---|---|---|
|0|`A`|`M`|
|1|`B`|`N`|
|2|`C`|`O`|
|...|...|...|
|13|`N`|`Z`|
|14|`O`|`A`|
|...|...|...|

`A → M` means encryption adds 12. Decryption subtracts 12, which is equivalent to adding 14 (since 26 − 12 = 14).

---

### Step 6 — Decrypt the Password

With the shift confirmed as +12 for encryption (−12 for decryption), apply the inverse with `tr`:

```bash
krypton2@krypton:/tmp/tmp.aBcDeFgH$ echo "OMQEMDUEQMEK" | tr 'A-Z' 'O-ZA-N'
```

Output:

```
<password>
```

That is the password for [Krypton Level 3 → 4](krypton-level-3-level-4.md).

---

## Key Takeaways

- **The Caesar cipher has no security against a chosen-plaintext attack.** Given an encryption oracle that uses the same key, encrypting any known plaintext immediately reveals the full substitution table. A single character is enough, `A` alone tells you the shift. The entire alphabet in one shot eliminates any ambiguity.
- **Setuid binaries write output under their effective UID.** `encrypt` runs as `krypton3`, so the output directory must be readable and writable by `krypton3`, hence `chmod 777`. This is the same setuid privilege mechanism exploited offensively in earlier levels, applied here in its intended, cooperative form.

---