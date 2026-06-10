> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                                                   |
| -------- | ------------------------------------------------------- |
| Host     | `leviathan.labs.overthewire.org`                        |
| Port     | `2223`                                                  |
| Username | `leviathan1`                                            |
| Password | *[Leviathan Level 0 → 1](leviathan-level-0-level-1.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `strings` — Extract printable strings from a binary file
- `ltrace` — Trace library calls made by a running program

> **Reference:** [ltrace man page](https://man7.org/linux/man-pages/man1/ltrace.1.html) · [strings man page](https://man7.org/linux/man-pages/man1/strings.1.html) · [setuid on Wikipedia](https://en.wikipedia.org/wiki/Setuid)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan1@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Survey the Home Directory

```bash
leviathan1@leviathan:~$ ls -la
```

Output:

```
-r-sr-x--- 1 leviathan2 leviathan1 7452 Jan  1 00:00 check
```

A single executable called `check`. Two things in the permissions field stand out immediately:

|Field|Value|Meaning|
|---|---|---|
|Owner|`leviathan2`|The binary belongs to the next level's user|
|Permissions|`-r-sr-x---`|The `s` in the owner-execute position means the setuid bit is set|
|Group|`leviathan1`|Readable and executable by our current user|

---

### Understand Setuid Binaries

Normally, a program runs with the privileges of the user who launched it. The **setuid bit** changes that: the program runs with the privileges of the file's **owner** instead, regardless of who executes it.

> Here, `check` is owned by `leviathan2` and has the setuid bit set. If it drops us into a shell, that shell runs as `leviathan2`, giving us read access to `/etc/leviathan_pass/leviathan2`. The goal is to satisfy whatever check the binary performs to reach that shell.

---

### Step 3 — Run the Binary

```bash
leviathan1@leviathan:~$ ./check
password:
```

It prompts for a password. Entering anything wrong produces:

```
Wrong password, Good Bye ...
```

---

### Step 4 — Try `strings` First

Before reaching for heavier tools, check whether the password is stored as a readable string inside the binary:

```bash
leviathan1@leviathan:~$ strings ./check
```

Among the output you'll see `love`, a plausible-looking candidate. Running the binary and entering `love` produces the same wrong-password message. `strings` shows every printable sequence in a binary, not only the ones that matter. It's a good first step, but not always conclusive.

---

### Understand `ltrace`

> `ltrace` intercepts calls to shared library functions as a program runs and prints them to the terminal, including the function name, its arguments, and the return value. Password checks almost always use a standard string-comparison function (`strcmp`, `strcasecmp`, `strncmp`). Running a binary under `ltrace` will expose both sides of that comparison: what you typed, and what the binary expected.

The syntax is:

```bash
ltrace ./program
```

It requires no special privileges and is purely observational — it doesn't modify the binary or its behaviour.

---

### Step 5 — Trace the Binary with `ltrace`

```bash
leviathan1@leviathan:~$ ltrace ./check
```

Enter any dummy value at the prompt. The output will include:

```
strcmp("d\n\n", "sex")                            = -1
```

`strcmp` is the C standard library function for comparing two strings. The first argument is what you typed; the second is what the binary is comparing it against. The password is `sex`.

---

### Step 6 — Run the Binary with the Correct Password

```bash
leviathan1@leviathan:~$ ./check
password: sex
```

A new shell prompt appears. Because `check` has the setuid bit, this shell is running as `leviathan2`. Confirm:

```bash
$ whoami
leviathan2
```

---

### Step 7 — Read the Password File

All Leviathan passwords are stored under `/etc/leviathan_pass/`, readable only by their respective user:

```bash
$ cat /etc/leviathan_pass/leviathan2
```

That is the password for [Leviathan Level 2 → 3](leviathan-level-2-level-3.md).

---

## Key Takeaways

- **`strings` is a first pass, not a final answer.** It surfaces readable text embedded in a binary, but binaries contain decoy strings, library names, and error messages alongside anything meaningful. `love` appearing in the output is a dead end, treat `strings` output as a lead to investigate, not a conclusion.
- **`ltrace` exposes what the binary actually does at runtime.** When a program compares your input to a secret value using a library function, `ltrace` shows you both sides of that comparison. It is one of the most efficient tools for reversing simple password checks without reading assembly.
- **The setuid bit is a privilege escalation primitive.** A setuid binary owned by another user runs with that user's rights. In CTFs and in the real world, misconfigured setuid binaries are a classic local privilege escalation vector.
- **`/etc/leviathan_pass/` is the canonical password store for this wargame.** Once you have a shell as the target user, however you obtained it, `cat /etc/leviathan_pass/<username>` retrieves the next password.

---