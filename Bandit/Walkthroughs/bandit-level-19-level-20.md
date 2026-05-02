> **Level Goal:** To gain access to the next level, you should use the setuid binary in the home directory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (`/etc/bandit_pass`), after you have used the setuid binary.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit19`                                                           |
| Password | *found in [Bandit Level 18 → Level 19](bandit-level-18-level-19.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls -l` — List directory contents with permissions
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [setuid on Wikipedia](https://en.wikipedia.org/wiki/Setuid)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit19@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 18 → Level 19](bandit-level-18-level-19.md).

---

### Step 2 — List the Home Directory

```bash
bandit19@bandit:~$ ls -l
-rwsr-x--- 1 bandit20 bandit19 14876 ... bandit20-do
```

There is one file: `bandit20-do`. The `-l` flag shows permissions in detail. The permission string `rwsr-x---` contains something new: the `s` in place of the usual `x` for the owner's execute bit. This is the **setuid** flag.

---

### Understand setuid

Normally when you run a program, it **executes** with **your** **user ID** and your permissions. [setuid](https://en.wikipedia.org/wiki/Setuid) changes this: when a binary has the setuid bit set, it **executes** with the **permissions** of the file's **owner** instead of the **user running it**.

Here `bandit20-do` is owned by `bandit20` and has the setuid bit set. That means when `bandit19` runs it, the process temporarily gains `bandit20`'s permissions — enough to read files that `bandit19` cannot.

The permission string broken down:

|Characters|Meaning|
|---|---|
|`rws`|Owner (`bandit20`) can read, write, and execute. The `s` means setuid is active — the binary runs as `bandit20` regardless of who launches it|
|`r-x`|Group (`bandit19`) can read and execute, but not write|
|`---`|Everyone else has no permissions|

---

### Step 3 — Run the Binary Without Arguments

```bash
bandit19@bandit:~$ ./bandit20-do
Run a command as another user.
  Example: ./bandit20-do id
```

The binary runs a command as `bandit20`. The usage is straightforward: pass the command you want to run as arguments.

---

### Step 4 — Read the Password File

`/etc/bandit_pass/bandit20` is readable only by `bandit20`. We use the setuid binary to run `cat` as `bandit20`:

```bash
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
<password>
```

Here is what each part does:

|Part|What it does|
|---|---|
|`./bandit20-do`|Runs the setuid binary, which executes as `bandit20`|
|`cat /etc/bandit_pass/bandit20`|The command to run — readable by `bandit20`, not by `bandit19`|

That is the password for [Bandit Level 20 → Level 21](bandit-level-20-level-21.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit19@bandit:~$ exit
```

---

## Key Takeaways

- The **setuid** bit causes a binary to **execute with the permissions of its owner**, not the user running it. It appears as `s` in the owner's execute position in `ls -l` output.
- **setuid binaries** are a **controlled way to grant users temporary** elevated **permissions** for a specific task — without giving them full access to another account.
- `ls -l` is **essential** for **spotting permission** bits like setuid. **Always check permissions** when a file behaves unexpectedly or when a level involves privilege escalation.
- **setuid is powerful** and a **common target in privilege escalation** attacks when misconfigured — a binary that runs as root with the **setuid** bit set can be **dangerous** if it allows **arbitrary command execution**.

---