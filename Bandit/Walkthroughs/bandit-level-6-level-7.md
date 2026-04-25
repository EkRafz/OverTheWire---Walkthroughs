> **Level Goal:** Visit the [Level 7](https://overthewire.org/wargames/bandit/bandit7.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit6`                                                     |
| Password | *found in [Bandit Level 5 → Level 6](bandit-level-5-level-6.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `find` — Search for files matching specific criteria
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 5 → Level 6](bandit-level-5-level-6.md).

---

### Step 2 — Understand the Problem

In the previous levels, `find` searched inside a specific directory (`inhere`). This time the level description says the file is stored **somewhere on the server** — meaning we need to search the entire filesystem. We do that by starting `find` from `/`, which is the **root of the filesystem** and the parent of every directory on the system.

```bash
bandit6@bandit:~$ find /
```

This alone would **flood the terminal** with every file on the system, including thousands we have no permission to read, producing a wall of `Permission denied` errors. We need to **filter** by the three properties given to us, and silence the noise.

---

### Step 3 — Consult `man find` to Find the Right Flags

We know three things about the file: who owns it (`bandit7`), what group it belongs to (`bandit6`), and how big it is (`33` bytes). The next question is: does `find` have flags for all three? Check the manual:

```bash
bandit6@bandit:~$ man find
```

Searching through the manual (use `/` to search inside `man`, then type the keyword and press `Enter`), the relevant entries are:

```
-user uname
       File is owned by user uname (numeric user ID allowed).

-group gname
       File belongs to group gname (numeric group ID allowed).

-size n[cwbkMG]
       File uses less than, more than or exactly n units of space,
       rounding up. The following suffixes can be used:

       'b'    for 512-byte blocks (this is the default if no suffix is used)
       'c'    for bytes
       'k'    for kibibytes (KiB, units of 1024 bytes)
       'M'    for mebibytes (MiB, units of 1024 * 1024 = 1048576 bytes)
       'G'    for gibibytes (GiB, units of 1024 * 1024 * 1024 bytes)
```

So:

- `-user bandit7` matches files owned by that user
- `-group bandit6` matches files belonging to that group
- `-size 33c` matches files that are exactly 33 bytes (`c` for bytes)

All three criteria are supported natively by `find` — no need for `grep` this time.

> **Tip — Navigating `man` pages:** Inside any `man` page: press `/` then type a keyword (e.g. `-user`) and hit `Enter` to jump to it. Press `n` to go to the next match. Press `q` to quit.

---

### Step 4 — Build the `find` Command

Here is the full command:

```bash
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

Each part does the following:

| Part             | What it does                                                        |
| ---------------- | ------------------------------------------------------------------- |
| `find /`         | Start searching from the root of the filesystem                     |
| `-user bandit7`  | Match only files owned by the user `bandit7`                        |
| `-group bandit6` | Match only files owned by the group `bandit6`                       |
| `-size 33c`      | Match only files that are exactly 33 bytes (`c` = bytes)            |
| `2>/dev/null`    | Silence all error messages so only real results are shown           |

---

### Step 5 — Understanding `2>/dev/null`

This part is new and worth understanding properly.

Every command in Linux has three standard data streams:

| Stream   | Number | Purpose                      |
| -------- | ------ | ---------------------------- |
| `stdin`  | `0`    | Input fed into the command   |
| `stdout` | `1`    | Normal output (results)      |
| `stderr` | `2`    | Error messages               |

By default, both `stdout` and `stderr` print to your terminal, which means errors appear mixed in with real results. When searching `/` as a non-root user, `find` will hit hundreds of directories it doesn't have permission to read, producing errors like:

```
find: '/root': Permission denied
find: '/etc/ssl/private': Permission denied
find: '/proc/tty/driver': Permission denied
...
```

The `>` operator **redirects** a stream to a destination instead of the terminal. `/dev/null` is a special file that **discards everything** written to it — it is sometimes called the **"black hole"** of Linux.

So `2>/dev/null` means: take stream 2 (errors) and redirect it into the void. Only `stdout` (the actual results) still prints to your terminal.

> **Tip — Redirecting stdout vs stderr:**
> - `>` or `1>` redirects normal output (stdout)
> - `2>` redirects errors (stderr)
> - `2>/dev/null` is the standard pattern for suppressing errors you don't care about

---

### Step 6 — Run the Command and Read the Result

```bash
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
```

One result. Read it:

```bash
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
```

The file contains the password for [Bandit Level 7 → Level 8](bandit-level-7-level-8.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit6@bandit:~$ exit
```

---

## Key Takeaways

- `/` is the root of the Linux filesystem — passing it to `find` searches the entire server, not just the current directory.
- `-user` and `-group` let you filter files by who owns them. Ownership is a core part of Linux permissions.
- Every command has three streams: `stdin` (0), `stdout` (1), and `stderr` (2). They can be redirected independently.
- `2>/dev/null` suppresses error messages by sending stream 2 to `/dev/null`, which discards everything written to it. This is a standard pattern any time `find` is run as a non-root user across the whole filesystem.

---
