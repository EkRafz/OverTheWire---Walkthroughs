> **Level Goal:** The password for the next level is stored in a file `readme` in the home directory. Unfortunately, someone has modified `.bashrc` to log you out when you log in with SSH.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit18`                                                           |
| Password | *found in [Bandit Level 17 → Level 18](bandit-level-17-level-18.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely, and run remote commands directly
- `cat` — Print file contents to the terminal

---

## Solution

### Understand the Problem

If you try to log in normally you will see:

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220
...
Byebye!
Connection to bandit.labs.overthewire.org closed.
```

`.bashrc` is a shell **configuration file** that runs automatically every time an **interactive bash session starts**. Whoever modified it **added a command that immediately exits the session** — so the shell starts, `.bashrc` runs, and you are kicked out before you can do anything.

The fix is to bypass the interactive shell entirely. SSH lets you **append a command to the end of the connection string**. When you do this, SSH **executes** that command on the **remote machine** and returns the output — without ever starting an interactive shell, so `.bashrc` is never sourced.

---

### Step 1 — Consult `man ssh` for Remote Command Execution

```bash
$ man ssh
```

The relevant excerpt:

```
ssh [options] [user@]hostname [command]

If a command is specified, it is executed on the remote host
instead of a login shell.
```

So `ssh user@host command` runs `command` on the remote machine and exits. No interactive shell, no `.bashrc`.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 2 — Read the File Directly Over SSH

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

Here is what each part does:

| Part                                               | What it does                                                |
| -------------------------------------------------- | ----------------------------------------------------------- |
| `ssh bandit18@bandit.labs.overthewire.org -p 2220` | Connects to the server as `bandit18`                        |
| `cat readme`                                       | The command to run on the remote machine instead of a shell |

The output is the contents of `readme` — printed directly to your local terminal. That is the password for [Bandit Level 19 → Level 20](bandit-level-19-level-20.md).

---

### Step 3 — Save the Password

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

There is no session to exit — the connection closes automatically once `cat readme` finishes.

---

## Key Takeaways

- `.bashrc` runs automatically at the start of every interactive bash session. **Malicious or misconfigured** `.bashrc` files can lock you out of a shell entirely.
- `ssh user@host command` **executes a single command** on the remote machine without starting an interactive shell — `.bashrc` is never sourced.
- This pattern is widely used in **scripts** and **automation** to run remote commands **without** needing a **full interactive session**.

---
