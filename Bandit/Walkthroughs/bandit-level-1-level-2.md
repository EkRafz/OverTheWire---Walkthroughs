> **Level Goal:** Visit the [Level 2](https://overthewire.org/wargames/bandit/bandit2.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit1`                                                     |
| Password | *found in [Bandit Level 0 → Level 1](bandit-level-0-level-1.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `pwd` — Print the current working directory
- `Ctrl + C` — Interrupt a running command and return to the prompt
- `Ctrl + Shift + C` — Copy selected text in the terminal

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 0 → Level 1](bandit-level-0-level-1.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit1@bandit:~$ ls
-
```

There is one file, and its name is `-`. This is where the challenge starts — `-` is a special character in the terminal and can't be used directly as a filename argument.

---

### Step 3 — The Problem with `cat -`

Your first instinct might be to run:

```bash
bandit1@bandit:~$ cat -
```

This won't read the file. In most Unix commands, `-` is a convention meaning _read from standard input_ (i.e., the keyboard), so the terminal just waits for you to type something — it's not an error, it's just doing the wrong thing.

> **Tip:** If you get stuck in this state, do **not** press `Esc`. Use **`Ctrl + C`** to interrupt the command and return to the prompt.

---

### Step 4 — Read the File Correctly

To read a file whose name is a special character, you need to give the terminal an explicit path to it instead of just the bare name. There are two ways to do this:

**Option A — Use the relative path** _(recommended)_

```bash
bandit1@bandit:~$ cat ./-
```

The `./` prefix means _"starting from the current directory"_. This tells `cat` you're referring to a file named `-` in the current directory, not invoking the standard input convention.

**Option B — Use the absolute path**

```bash
bandit1@bandit:~$ cat /home/bandit1/-
```

You can find your current directory at any time with `pwd`. This works, but is slower to type and less portable — `./` is the better habit.

The file contains the password for [Bandit Level 2 → Level 3](bandit-level-2-level-3.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit1@bandit:~$ exit
```

---

## Key Takeaways

- A filename of `-` is ambiguous to the shell — always use an explicit path or `./` to reference it.
- `cat -` reads from standard input, not from a file named `-`. This is a Unix convention, not a bug.
- `Ctrl + C` is your escape hatch when a command hangs unexpectedly.
- `./filename` is shorthand for the relative path of a file in your current directory and is generally preferable to typing the full absolute path.

---
