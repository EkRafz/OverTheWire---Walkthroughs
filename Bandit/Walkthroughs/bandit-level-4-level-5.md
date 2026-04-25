> **Level Goal:** Visit the [Level 5](https://overthewire.org/wargames/bandit/bandit5.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit4`                                                     |
| Password | *found in [Bandit Level 3 → Level 4](bandit-level-3-level-4.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cd` — Change the current directory
- `file` — Determine the type of a file
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 3 → Level 4](bandit-level-3-level-4.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit4@bandit:~$ ls
inhere
```

There is one directory named `inhere`. The password file is probably inside it.

---

### Step 3 — Navigate into the Directory

```bash
bandit4@bandit:~$ cd inhere
bandit4@bandit:~/inhere$
```

`cd` (change directory) moves you into the directory you specify. The prompt updates to show your new location: `~/inhere`.

---

### Step 4 — Identify the Files

Running `ls` shows ten files:

```bash
bandit4@bandit:~/inhere$ ls
-file00  -file02  -file04  -file06  -file08
-file01  -file03  -file05  -file07  -file09
```

All filenames start with `-`, which we know from [Bandit Level 1 → Level 2](bandit-level-1-level-2.md) needs to be handled with `./`. The challenge here is different though: there are ten files and only one contains human-readable text. Opening each one manually would be slow.

---

### Step 5 — Find the Human-Readable File

The `file` command inspects a file and reports what type of data it contains — it doesn't rely on the file extension, it reads the actual bytes. Running it on a single file looks like this:

```bash
bandit4@bandit:~/inhere$ file ./-file00
./-file00: data
```

`data` means binary — not human-readable. Rather than running this ten times, we can use a glob pattern to check all files at once:

```bash
bandit4@bandit:~/inhere$ file ./*
./-file00: data
./-file01: data
./-file02: data
./-file03: DOS executable (COM), start instruction 0x8c887e10 c3ee96c9
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```

> **Tip — How `./*` works:**
> - `./` — refers to the current directory, avoiding the `-` flag ambiguity
> - `*` — a **glob wildcard** that the shell expands to match every filename in the current directory
>
> The shell replaces `./*` with the full list of filenames before running the command, so `file ./*` is equivalent to running `file` on each file individually — just in one line.

Only `-file07` reports `ASCII text`, which is human-readable. Everything else is binary data.

---

### Step 6 — Read the File

```bash
bandit4@bandit:~/inhere$ cat ./-file07
```

The file contains the password for [Bandit Level 5 → Level 6](bandit-level-5-level-6.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit4@bandit:~/inhere$ exit
```

---

## Key Takeaways

- The `file` command identifies what a file actually contains, independent of its name or extension. It is the right tool when you need to distinguish binary files from human-readable ones.
- The glob wildcard `*` expands to all filenames in the current directory, letting you run a command across many files in one go.
- `./*` combines the relative path prefix with the wildcard — this is the correct form when filenames start with `-`, since bare `*` would expand to names beginning with `-` and confuse the shell into reading them as flags.
- `ASCII text` means human-readable; `data` means binary.

---
