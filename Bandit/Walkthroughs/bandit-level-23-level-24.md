> **Level Goal:** A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.
> 
> **Note:** This level requires you to create your own first shell script. This is a very big step and you should be proud of yourself when you beat this level!
> 
> **Note 2:** Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit23`                                                           |
| Password | *found in [Bandit Level 22 → Level 23](bandit-level-22-level-23.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `mkdir` — Create a directory
- `chmod` — Change file permissions
- `nano` / `vim` — Text editors for writing the shell script
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit23@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 22 → Level 23](bandit-level-22-level-23.md).

---

### Understand the Problem

The previous two levels had you _read_ a cron script to find where it put data. This level flips it: the cron job runs **any script you drop into a specific directory** — as `bandit24`. You write the script; cron executes it with `bandit24`'s permissions; your script copies the password somewhere you can read it.

Four things need to be true for this to work:

1. Your script must be **owned by `bandit23`** — the cron script checks the file owner with `stat` and skips anything not owned by you.
2. Your script must be **executable** — the cron script will not run a file without the execute bit set.
3. Your output location must be **writable by `bandit24`** and **readable by you** — a directory in `/tmp` that you create and make world-writable works.
4. Your script must be in place **before** the next cron tick — it runs every minute, so you have at most 60 seconds to wait.

---

### Step 2 — Look at `/etc/cron.d/`

```bash
bandit23@bandit:~$ cat /etc/cron.d/cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &>/dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &>/dev/null
```

Read the cron script:

```bash
bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash
shopt -s nullglob
myname=$(whoami)
cd /var/spool/"$myname"/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." ] && [ "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" "./$i")"
        if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
            timeout -s 9 60 "./$i"
        fi
        rm -rf "./$i"
    fi
done
```

---

### Understand the Script

|Line|What it does|
|---|---|
|`shopt -s nullglob`|If the directory is empty, `*` and `.*` expand to nothing instead of a literal `*` — the loop body never runs on an empty directory.|
|`myname=$(whoami)`|When cron runs this as `bandit24`, `myname` is `bandit24`.|
|`cd /var/spool/"$myname"/foo \| exit`|Changes into the drop zone, or exits immediately if the `cd` fails.|
|`for i in * .*`|Iterates over all files, including dotfiles. The `.` and `..` entries are explicitly excluded by the `if` condition below.|
|`owner="$(stat --format "%U" "./$i")"`|Gets the **owning username** of the file using `stat`.|
|`if [ "${owner}" = "bandit23" ] && [ -f "$i" ]`|Only executes the file if it is owned by `bandit23` **and** is a regular file. Files owned by anyone else are deleted but not run.|
|`timeout -s 9 60 "./$i"`|Runs the script, sending **SIGKILL** (signal 9) if it exceeds 60 seconds. Unlike the default SIGTERM, SIGKILL cannot be caught or ignored.|
|`rm -rf "./$i"`|Deletes the file unconditionally — whether it was executed or not.|

The owner check is the critical constraint: **only files owned by `bandit23` are executed**. This means you must copy the script into the drop zone while logged in as `bandit23` — the file will inherit your user ID as its owner. The drop zone is `/var/spool/bandit24/foo`.

---

### Step 3 — Create a Working Directory in `/tmp`

Your script needs somewhere to write the password — a location that `bandit24` can write to and you can read from. Create a directory in `/tmp` and make it world-writable:

```bash
bandit23@bandit:~$ mkdir /tmp/mydir24
bandit23@bandit:~$ chmod 777 /tmp/mydir24
```

`chmod 777` gives read, write, and execute permission to owner, group, and everyone else. This is intentionally permissive — `bandit24` (a different user) needs to be able to write into it.

---

### Step 4 — Write the Shell Script

Create the script in your `/tmp` directory first (recall that cron deletes it from the drop zone after execution — keeping a copy here lets you inspect it if something goes wrong):

```bash
bandit23@bandit:~$ nano /tmp/mydir24/script.sh
```

Enter the following:

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/mydir24/password
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano).

What the script does:

|Line|What it does|
|---|---|
|`#!/bin/bash`|Declares this is a bash script. Required — without a shebang, the shell does not know how to interpret the file.|
|`cat /etc/bandit_pass/bandit24 > /tmp/mydir24/password`|Reads `bandit24`'s password (accessible because the script runs _as_ `bandit24`) and writes it to a file you own the directory for.|

---

### Step 5 — Make the Script Executable

A file without the execute bit set is not a program — cron will skip it. Set the bit:

```bash
bandit23@bandit:~$ chmod +x /tmp/mydir24/script.sh
```

Verify with `ls -l`:

```bash
bandit23@bandit:~$ ls -l /tmp/mydir24/script.sh
-rwxrwxr-x 1 bandit23 bandit23 ... /tmp/mydir24/script.sh
```

The `x` bits confirm it is executable by everyone.

---

### Step 6 — Drop the Script into the Cron Directory

Copy (not move — keep your original) the script into the drop zone:

```bash
bandit23@bandit:~$ cp /tmp/mydir24/script.sh /var/spool/bandit24/foo/script.sh
```

Now wait. The cron job runs every minute. Within 60 seconds, `bandit24`'s cron script will find your file, execute it, and delete it. Your script will have written the password to `/tmp/mydir24/password`.

---

### Step 7 — Read the Password

Poll until the file appears:

```bash
bandit23@bandit:~$ cat /tmp/mydir24/password
<password>
```

If the file is not there yet, wait a few seconds and try again. If it never appears, check that the execute bit was set and that the script syntax is correct — re-drop it if needed.

That is the password for [Bandit Level 24 → Level 25](bandit-level-24-level-25.md).

---

### Step 8 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit23@bandit:~$ exit
```

---

## Key Takeaways

- A **shebang** (`#!/bin/bash`) at the top of a script tells the kernel which interpreter to use. Without it, the file is just text — not an executable program.
- **The execute bit** (`chmod +x`) is a separate gate from the shebang. Both are required: the shebang tells the system _how_ to run the file; the execute bit tells it the file _is allowed_ to be run.
- **`stat --format "%U" file`** returns the owning username of a file. The cron script uses this to only execute files owned by `bandit23` — a defence against arbitrary users dropping scripts into the shared drop zone. You must copy the script as `bandit23` for the owner check to pass.
- **World-writable directories** (`chmod 777`) are a deliberate tool here: you need a different user (`bandit24`) to write into a directory you created. In production systems, world-writable directories are a security risk — use them only when you understand exactly who will be writing there.
- **`cp` instead of `mv`** — keep a local copy of any script you drop into a directory that deletes its contents. The cron script uses `rm -rf` unconditionally, so even files it does not execute are deleted. If something goes wrong, you can re-drop without rewriting from scratch.
- The pattern across levels 21–24 is the same chain — **crontab → script → output** — with each level adding one more layer of indirection: **hardcoded** path, **computed** path, and finally, a **drop zone** where _you_ supply the script.

---
