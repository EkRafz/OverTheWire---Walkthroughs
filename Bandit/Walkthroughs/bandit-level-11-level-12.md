> **Level Goal:** The password for the next level is stored in the file `data.txt`, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit11`                                                           |
| Password | *found in [Bandit Level 10 → Level 11](bandit-level-10-level-11.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `tr` — Translate or replace characters
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [ROT13 on Wikipedia](https://en.wikipedia.org/wiki/ROT13)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit11@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 10 → Level 11](bandit-level-10-level-11.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit11@bandit:~$ ls
data.txt
```

One file: `data.txt`. Look at its contents:

```bash
bandit11@bandit:~$ cat data.txt
Gur cnffjbeq vf <scrambled>
```

The file is readable text, but the letters are wrong — every letter has been shifted by 13 positions. This is **ROT13**.

---

### Understand the Problem

[ROT13](https://en.wikipedia.org/wiki/ROT13) is a simple **letter substitution cipher** that shifts each letter **13 positions forward** in the alphabet, wrapping around at the end. Because the alphabet has 26 letters, **applying ROT13 twice returns the original text** — **encoding** and **decoding** are the **same operation**.

```
A → N    N → A
B → O    O → B
...
M → Z    Z → M
```

Numbers, punctuation, and spaces are left unchanged. We need to undo this shift to read the password.

The `tr` command translates characters — it takes a set of input characters and maps each one to a corresponding output character. By mapping every letter to its ROT13 equivalent, `tr` decodes the file in a single pass.

---

### Step 3 — Consult `man tr`

```bash
bandit11@bandit:~$ man tr
```

The relevant excerpt:

```
tr [OPTION]... SET1 [SET2]

Translate, squeeze, and/or delete characters from standard input,
writing to standard output.

SET1 and SET2 are sequences of characters. tr reads from stdin
and replaces each character in SET1 with the corresponding
character at the same position in SET2.
```

So `tr SET1 SET2` maps each character in SET1 to the character at the same position in SET2. If SET1 is `a-z` and SET2 is `n-za-m`, then `a` maps to `n`, `b` maps to `o`, and so on — exactly ROT13.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 4 — Run the Command

```bash
bandit11@bandit:~$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
The password is <password>
```

Here is what each part does:

| Part                         | What it does                                    |
| ---------------------------- | ----------------------------------------------- |
| `cat data.txt`               | Reads the file and sends it to `stdout`         |
| \|                           | Pipes the output into `tr` as its input         |
| `tr 'A-Za-z' 'N-ZA-Mn-za-m'` | Replaces every letter with its ROT13 equivalent |

Breaking down the character sets passed to `tr`:

|SET1|SET2|What it maps|
|---|---|---|
|`A-Z`|`N-ZA-M`|Uppercase: `A→N`, `B→O` ... `M→Z`, `N→A` ... `Z→M`|
|`a-z`|`n-za-m`|Lowercase: `a→n`, `b→o` ... `m→z`, `n→a` ... `z→m`|

The decoded output is a plain English sentence containing the password. That is the password for [Bandit Level 12 → Level 13](bandit-level-12-level-13.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit11@bandit:~$ exit
```

---

## Key Takeaways

- ROT13 **shifts** every letter **by 13 positions**. Because **13 + 13 = 26**, the same operation both **encodes** and **decodes**.
- `tr SET1 SET2` maps each character in SET1 to the character at the same position in SET2 — no loops, no scripting required.
- `'A-Za-z'` and `'N-ZA-Mn-za-m'` are the standard `tr` arguments for ROT13. The sets must be the same length, and position determines the mapping.
- `tr` reads from `stdin` only, so `cat file | tr ...` is the normal way to use it on a file.

---
