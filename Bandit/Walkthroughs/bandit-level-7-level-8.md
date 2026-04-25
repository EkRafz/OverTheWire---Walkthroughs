> **Level Goal:** Visit the [Level 8](https://overthewire.org/wargames/bandit/bandit8.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit7`                                                     |
| Password | *found in [Bandit Level 6 → Level 7](bandit-level-6-level-7.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `grep` — Filter output by a search term
- `wc` — Count lines, words, or bytes in a file
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 6 → Level 7](bandit-level-6-level-7.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit7@bandit:~$ ls
data.txt
```

There is one file: `data.txt`. Before reaching for `cat`, check how large it is:

```bash
bandit7@bandit:~$ wc -l data.txt
98567 data.txt
```

Nearly 100,000 lines. `wc -l` (word count, lines) is a quick sanity check before deciding how to approach a file — reading this one line by line in the terminal would be useless. We need to search it instead.

---

### Step 3 — Understand the Problem

The level tells us the password is on the same line as the word `millionth`. With nearly 100,000 lines to check, we need a tool that can find that one line without reading the entire file manually.

This is exactly what `grep` is for.

---

### Step 4 — Consult `man grep` to Understand the Tool

```bash
bandit7@bandit:~$ man grep
```

The key excerpt:

```
grep [OPTION...] PATTERNS [FILE...]

grep searches for PATTERNS in each FILE. PATTERNS is one or more
patterns separated by newline characters, and grep prints each line
that matches a pattern.
```

So `grep PATTERN FILE` reads a file line by line and prints only the lines that contain the pattern. That's all we need here — no special flags required.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 5 — Find the Password

There are **two ways** to search the file. Both produce identical output.

**Option A — `grep` directly on the file** _(recommended)_

```bash
bandit7@bandit:~$ grep "millionth" data.txt
millionth	<password>
```

`grep` opens and reads `data.txt` itself, with no other command involved. This is the standard and most concise way to search a single file.

**Option B — `cat` piped into `grep`**

```bash
bandit7@bandit:~$ cat data.txt | grep "millionth"
millionth	<password>
```

`cat data.txt` reads the entire file and sends all 98,567 lines to `stdout`. The `|` operator takes that stream and feeds it line by line into `grep "millionth"`, which prints only the matching lines. This form makes the data flow explicit and is easy to extend with additional filters — but adds an unnecessary process when searching a single file.

> **Tip — Which should you use?**
> Both are correct. `grep "millionth" data.txt` (Option A) is the cleaner choice for a single file. `cat data.txt | grep "millionth"` (Option B) is useful when you are chaining multiple commands and want to make the flow explicit.

---

### Step 6 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit7@bandit:~$ exit
```

---

## Key Takeaways

- `grep PATTERN FILE` searches a file line by line and prints only matching lines. It is the standard tool for finding a needle in a large text file.
- `cat FILE | grep PATTERN` is equivalent — `cat` feeds the file into `grep` via a pipe. Both forms are correct; `grep FILE` directly is the cleaner choice for a single file.
- `wc -l` tells you how many lines a file has — a useful first step before deciding how to approach it.
- Quoting the search pattern (`"millionth"`) is a good habit — it prevents the shell from misinterpreting special characters inside the pattern.

---
