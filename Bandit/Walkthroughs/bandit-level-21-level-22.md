> **Level Goal:** A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit21`                                                           |
| Password | *found in [Bandit Level 20 → Level 21](bandit-level-20-level-21.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cron` — Unix job scheduler
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit21@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 20 → Level 21](bandit-level-20-level-21.md).

---

### Understand the Problem

[Cron](https://en.wikipedia.org/wiki/Cron) is the standard **Unix job scheduler**. It runs **commands automatically** on a schedule — every minute, every hour, at boot, and so on. Scheduled jobs are defined in **crontab** files. System-wide crontabs live in `/etc/cron.d/`.

The level tells us a **cron job is running** as `bandit22`. If we can find out what that job does, we may be able to read its output.

---

### Step 2 — Look at `/etc/cron.d/`

```bash
bandit21@bandit:~$ ls /etc/cron.d/
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24
```

Each file configures a cron job for the corresponding user. Read the one for `bandit22`:

```bash
bandit21@bandit:~$ cat /etc/cron.d/cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &>/dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &>/dev/null
```

---

### Understand the Crontab Format

A crontab line has this structure:

```
minute hour day month weekday user command
```

The five time fields use `*` to mean "every". So `* * * * *` means the job runs every minute, every hour, every day. The `@reboot` line runs it once at system startup as well.

|Field|Value|Meaning|
|---|---|---|
|`* * * * *`|all five as `*`|Every minute|
|`bandit22`|user|Runs as `bandit22`|
|`/usr/bin/cronjob_bandit22.sh`|command|The script to run|
|`&>/dev/null`|redirection|Discards all output|

The script runs every minute as `bandit22`. Read it to see what it does:

---

### Step 3 — Read the Script

```bash
bandit21@bandit:~$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Every minute, this script:

1. Sets the permissions on a file in `/tmp` to `644` — readable by everyone
2. Writes `bandit22`'s password into that file

The file is world-readable and is being refreshed every minute. We can read it directly.

---

### Step 4 — Read the Password

```bash
bandit21@bandit:~$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
<password>
```

That is the password for [Bandit Level 22 → Level 23](bandit-level-22-level-23.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit21@bandit:~$ exit
```

---

## Key Takeaways

- **Cron** is the **Unix job scheduler**. It runs commands on a **fixed schedule** defined in **crontab files**.
- **System-wide cron jobs** are stored in `/etc/cron.d/`. Each line specifies a schedule, a user, and a command.
- The five time fields (`* * * * *`) map to **minute, hour, day, month, and weekday**. `*` means *"every"*.
- When investigating a system, cron jobs are **worth examining** — they **run as a specific user** on a schedule and often write output to predictable locations.
- Following the chain (crontab → script → output file) is the core skill this level teaches: **read the config, read the script, find where the data lands**.

---
