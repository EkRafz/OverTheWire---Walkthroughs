> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                                                   |
| -------- | ------------------------------------------------------- |
| Host     | `leviathan.labs.overthewire.org`                        |
| Port     | `2223`                                                  |
| Username | `leviathan3`                                            |
| Password | *[Leviathan Level 2 → 3](leviathan-level-2-level-3.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `ltrace` — Trace library calls made by a running program

> **Reference:** [ltrace man page](https://man7.org/linux/man-pages/man1/ltrace.1.html) · [strcmp man page](https://man7.org/linux/man-pages/man3/strcmp.3.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan3@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Survey the Home Directory

```bash
leviathan3@leviathan:~$ ls -la
```

Output:

```
-r-sr-x--- 1 leviathan4 leviathan3 10288 Jan  1 00:00 level3
```

A setuid binary called `level3`, owned by `leviathan4`. Same structure as the `check` binary in the previous level.

---

### Step 3 — Run the Binary

```bash
leviathan3@leviathan:~$ ./level3
Enter the password>
```

Enter anything wrong:

```
bzzzzzzzzap. WRONG
```

A password prompt, a rejection message. The pattern is identical to level 1→2.

---

### Step 4 — Trace the Binary with `ltrace`

```bash
leviathan3@leviathan:~$ ltrace ./level3
```

Enter a dummy value at the prompt. The full output:

```
strcmp("h0no33", "kakaka")                  = -1
printf("Enter the password> ")              = 20
fgets(Enter the password> test
"test\n", 256, 0xf7fab5c0)                 = 0xffffd24c
strcmp("test\n", "snlprintf\n")             = 1
puts("bzzzzzzzzap. WRONG")
+++ exited (status 0) +++
```

There are **two** `strcmp` calls here, not one.

---

### Understand the Decoy `strcmp`

> The first `strcmp("h0no33", "kakaka")` fires before the password prompt even appears. It is a decoy, a hardcoded comparison between two fixed strings that the binary performs on startup, presumably to mislead anyone skimming `ltrace` output. Neither string is the password. The tell is that neither argument matches your input: your dummy value (`"test\n"`) only appears in the **second** `strcmp`. The rule is to find the `strcmp` whose first argument matches what you typed, that is the one comparing your input against the secret. The second argument of that call is the password.

|`strcmp` call|First argument|Second argument|Relevant?|
|---|---|---|---|
|First|`"h0no33"`|`"kakaka"`|No — neither is your input|
|Second|`"test\n"`|`"snlprintf\n"`|Yes — first argument is what you typed|

The password is `snlprintf`.

---

### Step 5 — Run the Binary with the Correct Password

```bash
leviathan3@leviathan:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$
```

The binary drops a shell. Confirm the effective user:

```bash
$ whoami
leviathan4
```

---

### Step 6 — Read the Password File

```bash
$ cat /etc/leviathan_pass/leviathan4
```

That is the password for [Leviathan Level 4 → 5](leviathan-level-4-level-5.md).

---

## Key Takeaways

- **`ltrace` remains the right tool even when the binary looks identical to a previous one.**
- **Decoy comparisons are a basic obfuscation technique.** The first `strcmp` fires unconditionally before any user input is read.
- **`fgets` in `ltrace` output marks where user input enters the program.** The call `fgets("test\n", 256, ...)` shows both the buffer contents (your input) and the buffer size. Any `strcmp` after this call that takes your input string as an argument is the one to focus on.
- **The setuid → shell → `/etc/leviathan_pass/` chain is consistent across all Leviathan levels.** Once `whoami` confirms the new user, `cat /etc/leviathan_pass/<username>` always retrieves the next password.

---