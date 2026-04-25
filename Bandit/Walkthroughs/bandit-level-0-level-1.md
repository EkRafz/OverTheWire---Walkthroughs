> **Level Goal:** Visit the [Level 1](https://overthewire.org/wargames/bandit/bandit1.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                         |
| -------- | ----------------------------- |
| Host     | `bandit.labs.overthewire.org` |
| Port     | `2220`                        |
| Username | `bandit0`                     |
| Password | `bandit0`                     |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

---

### Step 2 — Enter the Password

You'll reach the login prompt:

```
bandit0@bandit.labs.overthewire.org's password:
```

Enter `bandit0`. The password won't be visible as you type — that's normal.

---

### Step 3 — Find the Password File

Once logged in, list the current directory:

```bash
bandit0@bandit:~$ ls
readme
```

There is one file named `readme`. Read it:

```bash
bandit0@bandit:~$ cat readme
```

The file contains the password for [Bandit Level 1 → Level 2](bandit-level-1-level-2.md).

---

### Step 4 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit0@bandit:~$ exit
```

---

## Key Takeaways

- `ls` lists the files in your current directory — always the first thing to run when you log in.
- `cat <filename>` prints a file's contents to the terminal.
- SSH passwords are invisible as you type — this is normal behaviour, not a bug.

---
