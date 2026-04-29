> **Level Goal:** The password for the next level is stored in the file `data.txt` in one of the few human-readable strings, preceded by several `=` characters.

---

## Connection Details

| Field    | Value                                                            |
| -------- | ---------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                    |
| Port     | `2220`                                                           |
| Username | `bandit9`                                                        |
| Password | *found in [Bandit Level 8 → Level 9](bandit-level-8-level-9.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `file` — Identify the type of a file
- `strings` — Extract human-readable strings from a binary file
- `grep` — Filter output by a search term
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 8 → Level 9](bandit-level-8-level-9.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit9@bandit:~$ ls
data.txt
```

One file: `data.txt`. Before assuming it is plain text, check what it actually is:

```bash
bandit9@bandit:~$ file data.txt
data.txt: data
```

`file` reports `data` — not `ASCII text`. This means the file is **binary**: it contains **arbitrary bytes**, not just printable characters. Opening it with `cat` would **flood the terminal with garbage** and likely corrupt your display. We need a different approach.

---

### Understand the Problem

The file is binary, but the level tells us the password is in one of the **few human-readable strings** inside it, preceded by several `=` characters. Two tools handle this together:

- `strings` — scans a binary file and extracts sequences of printable characters long enough to be readable text
- `grep` — filters those strings to show only the lines that match our pattern (`=`)

The key insight: most of the bytes in a binary file are non-printable noise. `strings` cuts through that noise and surfaces only the text-like sequences. `grep` then narrows those down to the one line we care about.


>**Why Can't We Just Use `cat` or `grep` Directly?**
>
>This is worth understanding. `cat data.txt` would **dump raw binary bytes** to your terminal — the output is mostly **non-printable characters** that render as garbled symbols and can leave the terminal in a broken state. `grep "=" data.txt` would work in the sense that it reads the file line by line, but **binary files** often **contain very few or no proper newlines**, so `grep` might treat the entire file as **one massive "line"** and produce unreliable output.
>
>The `strings` command solves both problems: it explicitly extracts only the printable sequences, ignores everything else, and outputs them as clean lines — a safe, predictable input for `grep`.

---

### Step 3 — Consult `man strings` and `man grep`

```bash
bandit9@bandit:~$ man strings
```

The relevant excerpt:

```
strings - print the sequences of printable characters in files

strings [option(s)] [file(s)]

For each file given, strings prints the printable character
sequences that are at least 4 characters long (or the number given
with the options below) and are followed by an unprintable
character.
```

By default, `strings` prints any **sequence of 4 or more printable characters**. That is all we need — no flags required.

```bash
bandit9@bandit:~$ man grep
```

The relevant excerpt:

```
grep [OPTION...] PATTERNS [FILE...]

grep searches for PATTERNS in each FILE and prints each line
that matches a pattern.
```

We will search for `=` as the pattern, which matches any line containing one or more `=` characters.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 4 — Run the Command

```bash
bandit9@bandit:~$ strings data.txt | grep "="
...
========== <password>
...
```

Here is what each part does:

| Part               | What it does                                                                                   |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| `strings data.txt` | Scans the binary file and outputs every sequence of printable characters (4+ chars by default) |
| \|                 | Pipes `strings`'s output into `grep` as its input                                              |
| `grep "="`         | Prints only lines that contain at least one `=` character                                      |

A few lines are printed. The password is on the line preceded by a run of `=` characters. That is the password for [Bandit Level 10 → Level 11](bandit-level-10-level-11.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit9@bandit:~$ exit
```

---

## Key Takeaways

- `file` identifies what a file actually contains — never assume a file is plain text without checking.
- `strings` extracts human-readable sequences from binary files. It is the standard first step when a binary file is known to contain embedded text.
- `strings FILE | grep PATTERN` is the standard pattern for finding a specific string inside a binary file.
- Running `cat` on a binary file can corrupt your terminal display. When `file` reports anything other than `ASCII text`, reach for `strings` first.

---
