> **Level Goal:** Visit the [Level 6](https://overthewire.org/wargames/bandit/bandit6.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit5`                                                     |
| Password | *found in [Bandit Level 4 → Level 5](bandit-level-4-level-5.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cd` — Change the current directory
- `find` — Search for files matching specific criteria
- `file` — Determine the type of a file
- `grep` — Filter output by a search term (optional, used to narrow results)
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 4 → Level 5](bandit-level-4-level-5.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit5@bandit:~$ ls
inhere
```

There is one directory named `inhere`. Navigate into it:

```bash
bandit5@bandit:~$ cd inhere
bandit5@bandit:~/inhere$
```

---

### Understand the Scale of the Problem

Running `ls` reveals 20 subdirectories:

```bash
bandit5@bandit:~/inhere$ ls
maybehere00  maybehere04  maybehere08  maybehere12  maybehere16
maybehere01  maybehere05  maybehere09  maybehere13  maybehere17
maybehere02  maybehere06  maybehere10  maybehere14  maybehere18
maybehere03  maybehere07  maybehere11  maybehere15  maybehere19
```

Each subdirectory contains multiple files:

```bash
bandit5@bandit:~/inhere$ ls maybehere00
-file1  -file2  -file3  spaces file1  spaces file2  spaces file3
```

Checking every file manually across all 20 directories is not realistic. We need to let the shell do the searching for us.

---

### Step 3 — Find the Right File with `find`

The `find` command searches a directory tree for files that match criteria you specify. The manual (`man find`) describes the flags we need:

| Flag               | What it does                                                        |
| ------------------ | ------------------------------------------------------------------- |
| `-type f`          | Match regular files only (not directories or links)                 |
| `-size 1033c`      | Match files that are exactly 1033 bytes (`c` = bytes)               |
| `! -executable`    | Match files that are **not** executable (`!` negates the condition) |
| `-exec file {} +`  | Run the `file` command on every match to show its type              |

Putting it together:

```bash
bandit5@bandit:~/inhere$ find . -type f ! -executable -size 1033c -exec file {} +
./maybehere07/.file2: ASCII text, with very long lines (1000)
```

There is only one result, so the `find` command alone is enough to identify the target.

---

### Step 4 — Filter with `grep` (Optional)

If `find` returned multiple results, you could narrow them down by piping the output into `grep` to show only human-readable files:

```bash
bandit5@bandit:~/inhere$ find . -type f ! -executable -size 1033c -exec file {} + | grep ASCII
./maybehere07/.file2: ASCII text, with very long lines (1000)
```

> **Tip — How the pipe `|` works:** The `|` operator takes the output of the command on its left and feeds it as input to the command on its right. Here, `find` produces a list of files with their types, and `grep ASCII` filters that list to show only lines containing the word `ASCII` — i.e. human-readable files.

This confirms the target: `./maybehere07/.file2`.

---

### Step 5 — Read the File

```bash
bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
```

The file contains the password for [Bandit Level 6 → Level 7](bandit-level-6-level-7.md).

---

### Step 6 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit5@bandit:~/inhere$ exit
```

---

## Key Takeaways

- `find` is the right tool when you need to locate a file by properties (size, type, permissions) rather than by name. It searches recursively through all subdirectories.
- The `!` operator negates a `find` condition — `! -executable` means "not executable".
- The `-size` flag uses suffixes to specify units: `c` = bytes, `k` = kilobytes, `M` = megabytes.
- `-exec file {} +` runs the `file` command on every result, showing what type of data each file contains.
- The pipe `|` chains commands together — the output of one becomes the input of the next. This is a core Unix pattern.
- `grep ASCII` filters any output to lines containing `ASCII`, isolating human-readable files from binary ones.

---
