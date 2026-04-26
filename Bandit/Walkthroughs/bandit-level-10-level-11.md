> **Level Goal:** The password for the next level is stored in the file `data.txt`, which contains base64 encoded data.

---

## Connection Details

| Field    | Value                                                              |
| -------- | ------------------------------------------------------------------ |
| Host     | `bandit.labs.overthewire.org`                                      |
| Port     | `2220`                                                             |
| Username | `bandit10`                                                         |
| Password | *found in [Bandit Level 9 → Level 10](bandit-level-9-level-10.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `base64` — **Encode** or **decode** base64 data
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [Base64 on Wikipedia](https://en.wikipedia.org/wiki/Base64)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit10@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 9 → Level 10](bandit-level-9-level-10.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit10@bandit:~$ ls
data.txt
```

One file: `data.txt`. Take a quick look at its contents:

```bash
bandit10@bandit:~$ cat data.txt
VGhlIHBhc3N3b3JkIGlzIGR...
```

The file contains a single line of seemingly random letters, numbers, and `=` characters. This is base64 encoded data — it looks like noise but is a **lossless text representation** of the original content. We need to decode it to read the password.

---

>**Understand the Problem**
>
>**[Base64](https://en.wikipedia.org/wiki/Base64)** is an **encoding scheme** that represents binary data using only 64 printable ASCII characters (`A–Z`, `a–z`, `0–9`, `+`, `/`) with `=` used as padding at the end. It is **not encryption**, there is no key and no secret. Anyone with the encoded string can decode it instantly. It is used to safely **transmit binary data** over channels that only handle text (email attachments, for example).
>
>The `base64` command **handles** both **encoding** and **decoding**. The `-d` flag switches it to decode mode.

---

### Step 4 — Consult `man base64`

```bash
bandit10@bandit:~$ man base64
```

The relevant excerpt:

```
base64 [OPTION]... [FILE]

Base64 encode or decode FILE, or standard input, to standard output.

-d, --decode
        decode data
```

With `-d`, `base64` reads the encoded input and writes the original decoded content to `stdout`. No other flags are needed.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 5 — Run the Command

```bash
bandit10@bandit:~$ base64 -d data.txt
The password is <password>
```

Here is what each part does:

|Part|What it does|
|---|---|
|`base64`|Invokes the base64 tool|
|`-d`|Switches to decode mode (default is encode)|
|`data.txt`|The file to read encoded input from|

The decoded output is a plain English sentence containing the password. That is the password for [Bandit Level 11 → Level 12](bandit-level-11-level-12.md).

---

### Step 6 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit10@bandit:~$ exit
```

---

## Key Takeaways

- Base64 is an **encoding** scheme, not encryption. It has no key and provides no security — it only converts data into a portable ASCII-safe format.
- `base64 -d FILE` **decodes** a **base64-encoded file** and writes the original content to `stdout`.
- The trailing `=` or `==` characters in a base64 string are padding to make the total length a multiple of 4 — they are not part of the data.
- Base64 encoded strings use only `A–Z`, `a–z`, `0–9`, `+`, `/`, and `=`. If you see a file with exactly these characters, base64 is the likely encoding.

---
