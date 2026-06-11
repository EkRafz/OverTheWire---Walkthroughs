> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                                                   |
| -------- | ------------------------------------------------------- |
| Host     | `leviathan.labs.overthewire.org`                        |
| Port     | `2223`                                                  |
| Username | `leviathan6`                                            |
| Password | *[Leviathan Level 5 → 6](leviathan-level-5-level-6.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `ltrace` — Trace library calls made by a running program
- `for` loop — Iterate over a range of values in Bash

> **Reference:** [seq man page](https://man7.org/linux/man-pages/man1/seq.1.html) · [Bash for loops](https://www.gnu.org/software/bash/manual/bash.html#Looping-Constructs)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan6@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Survey the Home Directory

```bash
leviathan6@leviathan:~$ ls -la
```

Output:

```
-r-sr-x--- 1 leviathan7 leviathan6 7452 Jan  1 00:00 leviathan6
```

A setuid binary called `leviathan6`, owned by `leviathan7`.

---

### Step 3 — Run the Binary

```bash
leviathan6@leviathan:~$ ./leviathan6
usage: ./leviathan6 <4 digit code>
leviathan6@leviathan:~$ ./leviathan6 0000
Wrong
```

It takes a 4-digit PIN as a command-line argument and rejects incorrect input with `Wrong`. The search space is 0000–9999: 10,000 possibilities.

---

### Step 4 — Try `ltrace`

```bash
leviathan6@leviathan:~$ ltrace ./leviathan6 0000
```

The output shows `atoi` being called to convert the argument to an integer, but no `strcmp` or similar comparison against a visible string. The PIN is stored as a number and compared numerically, so `ltrace` alone doesn't expose it — unlike previous levels where a string comparison function showed both sides.

---

### Understand the Brute Force Approach

The PIN is a 4-digit code: the search space is 0000–9999, exactly 10,000 possibilities. A Bash loop can try all of them in sequence and stop the moment the binary opens a shell rather than printing `Wrong`.

> `seq -w 0000 9999` generates every number from 0 to 9999 zero-padded to four digits. The loop passes each value to the binary; `&&` means "if the previous command succeeded, run the next one" — so `break` fires only when the binary exits with status 0, which happens only on the correct PIN.

---

### Step 5 — Run the Brute Force Loop

```bash
leviathan6@leviathan:~$ for i in $(seq -w 0000 9999); do ./leviathan6 $i; done
```

Breaking down the command:

| Component          | Purpose                                                 |
| ------------------ | ------------------------------------------------------- |
| `seq -w 0000 9999` | Generates zero-padded values from `0000` to `9999`      |
| `./leviathan6 $i`  | Runs the binary with each candidate PIN                 |

The loop runs for a minute or two. When the correct PIN is reached the binary opens a shell instead of exiting, `break` fires, and you're dropped into that shell.

---

### Step 6 — Confirm the Shell

A shell opens. Confirm the effective user:

```bash
$ whoami
leviathan7
```

---

### Step 7 — Read the Password File

```bash
$ cat /etc/leviathan_pass/leviathan7
```

That is the password for this level , the final level.

---

## Key Takeaways

- **A small, bounded keyspace makes brute force legitimate.** A 4-digit PIN has exactly 10,000 possibilities. A Bash loop exhausts that in under two minutes without any knowledge of the binary's internals. Brute force is not always the right tool, but when the keyspace is small and enumeration is cheap, it is the simplest path.
- **`seq -w` is the right tool for zero-padded sequences.** Plain `{0..9999}` in Bash generates numbers without leading zeros, so `7` instead of `0007`. The `-w` flag in `seq` pads all values to the same width as the largest number, which is required when the binary expects exactly four digits.
---