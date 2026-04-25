> **Level Goal:** Visit the [Level 4](https://overthewire.org/wargames/bandit/bandit4.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit3`                                                     |
| Password | *found in [Bandit Level 2 → Level 3](bandit-level-2-level-3.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `ls -la` — List all files including hidden ones, in long format
- `ll` — Shorthand alias for `ls -la` (may not be available on all systems)
- `cd` — Change the current directory
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 2 → Level 3](bandit-level-2-level-3.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit3@bandit:~$ ls
inhere
```

There is one directory named `inhere`. The password file is probably inside it.

---

### Step 3 — Navigate into the Directory

Use `cd` to move into it:

```bash
bandit3@bandit:~$ cd inhere
bandit3@bandit:~/inhere$
```

`cd` (change directory) moves you into the directory you specify. Notice the prompt updates to show your new location: `~/inhere`.

---

### Step 4 — The Problem with `ls`

Running a plain `ls` shows nothing:

```bash
bandit3@bandit:~/inhere$ ls
bandit3@bandit:~/inhere$
```

The directory appears empty, but it isn't. On Unix systems, any file or directory whose name begins with a `.` is considered **hidden** and is not shown by `ls` by default.

---

### Step 5 — List Hidden Files

To reveal hidden files, pass the `-a` flag (all) to `ls`. Adding `-l` as well gives a detailed long-format view that shows permissions, ownership, size, and modification time — useful context for a CTF:

```bash
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Apr  3 15:17 ./
drwxr-xr-x 3 root    root    4096 Apr  3 15:17 ../
-rw-r----- 1 bandit4 bandit3   33 Apr  3 15:17 ...Hiding-From-You
```

> **Tip:** On many systems `ll` is a shell alias for `ls -la` and produces the same output. However, it is not available everywhere — `ls -la` is the portable version and the one to rely on.

Here is what each column in the output means:

| Column             | Example            | Meaning                                                                                      |
| ------------------ | ------------------ | -------------------------------------------------------------------------------------------- |
| Type & permissions | `-rw-r-----`       | `-` = file, `d` = directory, `l` = link; then read/write/execute for owner, group, others   |
| Hard links         | `1`                | Number of hard links pointing to this file                                                   |
| Owner              | `bandit4`          | User who owns the file                                                                       |
| Group              | `bandit3`          | Group with access to the file                                                                |
| Size (bytes)       | `33`               | File size in bytes (`-h` flag adds human-readable units like K/M/G)                         |
| Modification time  | `Apr 3 15:17`      | When the file was last modified                                                              |
| Name               | `...Hiding-From-You` | The filename — note the `.` prefix marking it as hidden                                    |

The hidden file is named `...Hiding-From-You`.

---

### Step 6 — Read the File

```bash
bandit3@bandit:~/inhere$ cat ./...Hiding-From-You
```

The `./` prefix avoids any ambiguity, and is a good habit whenever a filename starts with special characters. The file contains the password for [Bandit Level 4 → Level 5](bandit-level-4-level-5.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit3@bandit:~/inhere$ exit
```

---

## Key Takeaways

- Files and directories whose names begin with `.` are hidden from `ls` by default.
- `ls -la` (or its alias `ll`) reveals hidden files and provides detailed metadata for each entry. Use `ls -la` when you need portability.
- `cd <directory>` moves you into a directory; your prompt updates to reflect where you are.
- Even hidden files with unusual names like `...Hiding-From-You` can be read with `cat ./filename` — the `./` ensures the shell treats the name as a path, not a flag.

---
