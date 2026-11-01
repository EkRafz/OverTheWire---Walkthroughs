> **Level Goal:** At this moment, level 34 does not exist yet.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit33`                                                           |
| Password | *found in [Bandit Level 32 → Level 33](bandit-level-32-level-33.md)* |

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh bandit33@bandit.labs.overthewire.org -p 2220
```

---

### Step 2 — Read the README

```bash
bandit33@bandit:~$ ls
README.txt
bandit33@bandit:~$ cat README.txt
Congratulations on solving the last level of this game!
At this moment, there are no more levels to play in this game.
However, we are constantly working on new levels and will most
likely expand this game with more levels soon.

Keep an eye out for an announcement on our usual communication
channels! In the meantime, you could play some of our other
wargames. If you have an idea for an awesome new level, please
let us know!
```

That's it. You've completed Bandit.

---

## What You've Covered

Across 33 levels, Bandit introduced the core toolkit of Linux command-line and security fundamentals:

| Category            | Skills                                                                       |
| ------------------- | ---------------------------------------------------------------------------- |
| **File system**     | `ls`, `cat`, `find`, hidden files, permissions, `chmod`, setuid              |
| **Text processing** | `grep`, `sort`, `uniq`, `strings`, `diff`, `tr`, `cut`, `base64`             |
| **Compression**     | `gzip`, `bzip2`, `tar`, `xxd`                                                |
| **Networking**      | `ssh`, `nc`, `openssl`, `nmap`, ports, localhost                             |
| **Shell**           | Pipes, redirection, job control, `$0`, special variables                     |
| **Scripting**       | Writing shell scripts, cron, drop zones                                      |
| **Git**             | `clone`, `log`, `show`, `branch`, `checkout`, `tag`, `add`, `commit`, `push` |
| **Escapes**         | Restricted shells, pager escapes, editor escapes, uppercase shell            |

---
