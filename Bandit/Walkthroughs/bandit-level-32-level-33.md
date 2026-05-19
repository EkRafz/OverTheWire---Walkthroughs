> **Level Goal:** After all this git stuff, it's time for another escape. Good luck!

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit32`                                                           |
| Password | *found in [Bandit Level 31 → Level 32](bandit-level-31-level-32.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `$0` — Special shell variable that expands to the current shell's name
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh bandit32@bandit.labs.overthewire.org -p 2220
```

---

### Understand the Problem

Logging in drops you into what the prompt calls the UPPERCASE SHELL:

```
WELCOME TO THE UPPERCASE SHELL
>>
```

Whatever you type is converted to uppercase before being executed:

```
>> ls
sh: 1: LS: not found
>> cat /etc/bandit_pass/bandit32
sh: 1: CAT: not found
>> whoami
sh: 1: WHOAMI: not found
```

Every command fails because Unix commands are lowercase — `LS`, `CAT`, and `WHOAMI` do not exist. Typing any standard command is useless.

The constraint is specifically on **alphabetic characters** — they are uppercased. The escape is to find something that does not contain any alphabetic characters at all.

---

### Understand $0

In any shell, a set of **special variables** are set automatically and cannot be uppercased because they contain no letters. The most useful here is `$0`.

`$0` expands to the **name of the currently running shell or script**. When you are in an interactive shell, `$0` is the path to the shell binary itself — for example `/bin/sh` or `sh`.

Critically: `$0` contains no alphabetic characters of its own. The `$` and `0` are not letters, so the uppercase shell has nothing to convert. When evaluated, `$0` expands to the shell path and the shell executes it — launching a new, normal shell as a child process.

---

### Step 2 — Escape with $0

At the UPPERCASE SHELL prompt, type:

```
>> $0
```

This spawns a new `/bin/sh` process. You now have a normal shell prompt:

```
$
```

Verify:

```
$ whoami
bandit33
```

The uppercase shell was running as `bandit33`. Escaping it gives you a normal shell with the same user — no privilege change needed, you were already `bandit33`.

---

### Step 3 — Read the Password

```bash
$ cat /etc/bandit_pass/bandit33
<password>
```

That is the password for [Bandit level 33 → level 34](bandit-level-33-level-34.md).

---

### Step 4 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
$ exit
```

---

## Key Takeaways

- **Restricted shells constrain what you can type**, not necessarily what the shell _can_ do. The uppercase shell transforms input before execution — but only alphabetic characters. Anything without letters passes through untouched.
- **`$0`** is a shell special variable that expands to the running shell's name. Because it contains no letters, it survives the uppercase transformation and executes as-is, spawning a normal child shell.
- **Shell special variables** (`$0`, `$1`, `$$`, `$?`, `$#`) are worth knowing precisely because they contain no alphabetic characters. They are useful anywhere alphabetic input is restricted or filtered.
- This escape is a close relative of the `vi` escape in Level 25 → 26: both work by finding **interactive surface within the restriction itself** rather than trying to break out by force. The uppercase shell is still a shell — it still evaluates `$0`.
- **Restricted environments are only as strong as their constraints are complete.** A filter that covers commands but not shell syntax leaves the shell's own evaluation machinery reachable.

---
