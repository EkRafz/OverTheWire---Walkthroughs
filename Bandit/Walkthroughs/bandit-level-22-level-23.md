> **Level Goal:** A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.
> 
> **Note:** Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit22`                                                           |
| Password | *found in [Bandit Level 21 → Level 22](bandit-level-21-level-22.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `echo` — Print a string to the terminal
- `md5sum` — Compute an MD5 hash
- `cut` — Extract fields from a line of text
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit22@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 21 → Level 22](bandit-level-21-level-22.md).

---

### Understand the Problem

Like the previous level, a cron job is running as `bandit23`. The goal is the same: **find** the cron job, **read** the script it runs, and **figure** out where it puts the password.

The difference this time is that the output file is not **hardcoded** — its name is **computed at runtime** by the script. You need to **read** the script, **understand** the computation, and **reproduce** it yourself to find the filename.

---

### Step 2 — Look at `/etc/cron.d/`

```bash
bandit22@bandit:~$ ls /etc/cron.d/
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24
```

Read the entry for `bandit23`:

```bash
bandit22@bandit:~$ cat /etc/cron.d/cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh &>/dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh &>/dev/null
```

The script `/usr/bin/cronjob_bandit23.sh` runs every minute as `bandit23`. Output is discarded with `&>/dev/null`, so we cannot observe it directly. Read the script instead.

---

### Step 3 — Read the Script

```bash
bandit22@bandit:~$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

---

### Understand the Script

Work through it line by line:

| Line                                                  | What it does                                                                                                         |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `myname=$(whoami)`                                    | Stores the name of the user running the script. When cron runs this, `myname` is `bandit23`.                         |
| `echo I am user $myname \| md5sum \| cut -d ' ' -f 1` | Hashes the string `"I am user bandit23"` with MD5, then extracts just the hash (the first field, cutting on spaces). |
| `cat /etc/bandit_pass/$myname > /tmp/$mytarget`       | Writes `bandit23`'s password into `/tmp/<hash>`.                                                                     |

The output file is `/tmp/<md5 hash of "I am user bandit23">`. The file is written by `bandit23`, but nothing in the script restricts who can read it — permissions will be world-readable (the script does not call `chmod` to lock it down).

To find the file, reproduce the hash computation yourself.

---

### Step 4 — Compute the Target Filename

Run the same pipeline the script runs, but substitute `bandit23` for `$myname` explicitly — because you are `bandit22`, `whoami` would give the wrong result:

```bash
bandit22@bandit:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349
```

The password file is `/tmp/8ca319486bfbbc3663ea0fbe81326349`.

---

### Understand `md5sum` and `cut`

**`md5sum`** computes the MD5 hash of its input and prints it in the format:

```
<32-character hex hash>  -
```

The trailing `-` is the filename (stdin is represented as `-`). We only want the hash itself, so we pipe through `cut`.

**`cut -d ' ' -f 1`** splits the line on spaces (`-d ' '`) and returns only the first field (`-f 1`) — the hash.

---

### Step 5 — Read the Password

```bash
bandit22@bandit:~$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
<password>
```

That is the password for [Bandit Level 23 → Level 24](bandit-level-23-level-24.md).

---

### Step 6 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit22@bandit:~$ exit
```

---

## Key Takeaways

- When a cron script writes to a **computed filename**, read the script and **reproduce the computation** yourself — the filename is not secret, just not obvious.
- `whoami` returns the current user. When a script runs under cron, `whoami` returns the user cron runs it as — not you. **Substitute the correct username manually** when reproducing the computation.
- `echo "string" | md5sum | cut -d ' ' -f 1` is a compact pipeline to hash a string and extract just the hex digest. Recognising this pattern is useful — it appears in real scripts that generate predictable-but-opaque filenames.
- **Reading other people's shell scripts** is a practical skill. Follow each variable assignment, trace what each pipeline produces, and work out the final state. This level is a controlled exercise in exactly that.
- The previous level's script hardcoded its output path. This one computes it — a small increase in complexity, but the method is the same: **read the config, read the script, reproduce the logic, find where the data lands**.

---
