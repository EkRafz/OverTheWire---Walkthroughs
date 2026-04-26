> **Level Goal:** The password for the next level is stored in the file `data.txt`, which is a hexdump of a file that has been repeatedly compressed.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit12`                                                           |
| Password | *found in [Bandit Level 11 → Level 12](bandit-level-11-level-12.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `mktemp -d` — Create a temporary working directory with a unique name
- `cp` — Copy a file
- `mv` — Rename or move a file
- `xxd` — Create or reverse a hexdump
- `file` — Identify the type of a file
- `gzip` — Compress or decompress gzip files
- `bzip2` — Compress or decompress bzip2 files
- `tar` — Extract tar archives
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [Hex dump on Wikipedia](https://en.wikipedia.org/wiki/Hex_dump)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit12@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 11 → Level 12](bandit-level-11-level-12.md).

---

### Step 2 — Create a Working Directory

Your **home directory** is **read-only** on this level. The level tells us to work under `/tmp`. Rather than guessing a unique name, use `mktemp -d` which **creates a temporary directory** with a guaranteed unique name:

```bash
bandit12@bandit:~$ mktemp -d
/tmp/tmp.aB3xQr9kLm
```

Copy `data.txt` into it and move there:

```bash
bandit12@bandit:~$ cp data.txt /tmp/tmp.aB3xQr9kLm/
bandit12@bandit:~$ cd /tmp/tmp.aB3xQr9kLm
```

---

### Step 3 — Understand the Problem

The file `data.txt` is a [hexdump](https://en.wikipedia.org/wiki/Hex_dump) — a **human-readable representation of binary data** where **each byte** is printed as **two hexadecimal digits**. Before we can do anything useful, we need to **reverse the hexdump back into binary**. Then we face the real challenge: the binary file has been **compressed multiple times**, using a mix of `gzip`, `bzip2`, and `tar`. We do not know how many layers there are or in what order.

The strategy is a loop:

1. Run `file` to identify what the current file actually is
2. Rename the file to match its true type (so decompression tools accept it)
3. Decompress it
4. Repeat until `file` reports plain text

---

### Step 4 — Reverse the Hexdump

```bash
bandit12@bandit:/tmp/tmp.aB3xQr9kLm$ man xxd
```

The relevant excerpt:

```
xxd [options] [infile [outfile]]

xxd creates a hexdump of a given file or standard input.
It can also convert a hexdump back to its original binary form.

-r, --reverse
        Reverse operation: convert (or patch) hexdump into binary.
```

Run it:

```bash
bandit12@bandit:/tmp/tmp.aB3xQr9kLm$ xxd -r data.txt > data.bin
```

| Part         | What it does                                                   |
| ------------ | -------------------------------------------------------------- |
| `xxd -r`     | Reverses a hexdump back to binary                              |
| `data.txt`   | The hexdump file to read                                       |
| `> data.bin` | Redirects the binary output into a new file, `.bin` for binary |

---

### Step 5 — Identify and Decompress Repeatedly

Now run `file` on the binary to see what it is:

```bash
bandit12@bandit:/tmp/tmp.aB3xQr9kLm$ file data.bin
data.bin: gzip compressed data, was "data2.bin", ...
```

The output tells you exactly **what to do next**. Use the table below to match `file`'s output to the right decompression command. Repeat this process — run `file`, decompress, run `file` again — until you reach plain text.

---

### Step 6 — Decompression Reference

#### gzip

```
file output: gzip compressed data
```

`gzip` requires the file to have a `.gz` extension. Rename it first, then decompress:

```bash
$ mv data.bin data.gz
$ gzip -d data.gz
```

`gzip -d` decompresses and removes the `.gz` file, leaving the extracted file behind. Check `man gzip`:

```
-d, --decompress
        Decompress the compressed file.
```

---

#### bzip2

```
file output: bzip2 compressed data
```

Same pattern — `bzip2` requires a `.bz2` extension:

```bash
$ mv data.bin data.bz2
$ bzip2 -d data.bz2
```

Check `man bzip2`:

```
-d, --decompress
        Force decompression.
```

---

#### tar (gzip-compressed)

```
file output: POSIX tar archive (GNU)
```

`tar` archives often contain one or more files. Extract with:

```bash
$ mv data.bin data.tar
$ tar -xf data.tar
```

Check `man tar`:

```
-x, --extract
        Extract files from an archive.

-f, --file=ARCHIVE
        Use archive file ARCHIVE.
```

`tar -xf` extracts the archive contents into the current directory. Run `ls` after extracting to see what file was produced, then run `file` on that file to continue.

---

### Step 7 — Keep Going Until You Reach Plain Text

After each decompression step, run `file` on the resulting file. The name of the output file will vary — use `ls` after each step to see what was produced. Continue until `file` reports:

```
data[X]: ASCII text
```

At that point, read the file:

```bash
$ cat data[X]
The password is <password>
```

That is the password for [Bandit Level 13 → Level 14](bandit-level-13-level-14.md).

---

### Step 8 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit12@bandit:/tmp/tmp.aB3xQr9kLm$ exit
```

---

## Key Takeaways

- A **hexdump** is a **text representation of binary data**. `xxd -r` **reverses** it back to binary.
- `file` **identifies** a file's **true type** regardless of its name or extension — always run it before deciding what to do next.
- Compression tools (`gzip`, `bzip2`) **require** the **correct file extension** to operate. Use `mv` to rename before decompressing.
- `tar -xf` **handles both gzip and bzip2** compressed archives without needing extra flags.
- `mktemp -d` is the **safest way to create a temporary working directory** — it guarantees a unique name and avoids collisions with other users on the same system.
- The `>` operator redirects `stdout` to a file. `xxd -r data.txt > data.bin` writes the binary output to `data.bin` rather than printing it to the terminal.

---
