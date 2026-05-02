> **Level Goal:** Good job getting a shell! Now hurry and grab the password for bandit27!

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit26`                                                           |
| Password | *found in [Bandit Level 25 → Level 26](bandit-level-25-level-26.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls -l` — List directory contents with permissions
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Get a Shell as bandit26

You cannot SSH into `bandit26` normally — its login shell is `/usr/bin/showtext`, which exits immediately unless `more` is forced into interactive mode. Follow the escape chain from [Bandit Level 25 → Level 26](bandit-level-25-level-26.md) to get a working bash shell first:

1. Shrink your terminal to around 5 lines tall.
2. `ssh -i bandit26.sshkey bandit26@localhost -p 2220` from the `bandit25` prompt.
3. When `more` pauses, press `v` to open `vi`.
4. In `vi`: `:set shell=/bin/bash` then `:shell`.

You now have an interactive shell as `bandit26`.

---

### Step 2 — List the Home Directory

```bash
bandit26@bandit:~$ ls -l
-rwsr-x--- 1 bandit27 bandit26 14876 ... bandit27-do
-rw-r----- 1 bandit26 bandit26   258 ... text.txt
```

There are two files. `text.txt` is the file `more` was displaying. The interesting one is `bandit27-do`: a setuid binary owned by `bandit27` with the setuid bit set (`rws` in the owner position).

This is the same pattern as [Bandit Level 19 → Level 20](bandit-level-19-level-20.md). The binary runs a command as its owner (`bandit27`) regardless of who executes it.

---

### Step 3 — Run the Binary Without Arguments

```bash
bandit26@bandit:~$ ./bandit27-do
Run a command as another user.
  Example: ./bandit27-do id
```

---

### Step 4 — Read the Password File

```bash
bandit26@bandit:~$ ./bandit27-do cat /etc/bandit_pass/bandit27
<password>
```

`/etc/bandit_pass/bandit27` is readable only by `bandit27`. The setuid binary executes `cat` as `bandit27`, so the read succeeds.

That is the password for [Bandit Level 27 → Level 28](bandit-level-27-level-28.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit26@bandit:~$ exit
```

---

## Key Takeaways

- The setuid binary here is identical in behaviour to the one in Level 19 → 20. Recognising a pattern you have seen before and applying it immediately is a core skill — not every level introduces something new.
- **Getting a shell is not always the final step.** Here the shell from the previous level is the _starting point_. Levels can chain: the escape you worked for in Level 25 → 26 is the prerequisite for this one.
- If you close your terminal or lose the shell, you have to repeat the `more` → `vi` → `:shell` escape from Level 25 → 26. The password you just found lets you skip straight to `ssh bandit26@...` next time — but you will still need to shrink the terminal and repeat the escape chain to get a usable shell.

---
