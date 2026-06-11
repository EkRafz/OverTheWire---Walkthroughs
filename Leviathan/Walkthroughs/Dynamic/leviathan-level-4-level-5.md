> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                                                   |
| -------- | ------------------------------------------------------- |
| Host     | `leviathan.labs.overthewire.org`                        |
| Port     | `2223`                                                  |
| Username | `leviathan4`                                            |
| Password | *[Leviathan Level 3 → 4](leviathan-level-3-level-4.md)* |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `ltrace` — Trace library calls made by a running program
- `echo` — Print a string to standard output
- `perl` — Run a one-liner Perl script

> **Reference:** [ASCII table](https://www.asciitable.com/) · [pack in Perl](https://perldoc.perl.org/functions/pack) · [ltrace man page](https://man7.org/linux/man-pages/man1/ltrace.1.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan4@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Survey the Home Directory

```bash
leviathan4@leviathan:~$ ls -la
```

Output:

```
dr-xr-x---   2 root leviathan4 4096 Apr  3 15:19 .trash
```

A hidden directory called `.trash`, readable and executable by `leviathan4` but owned by `root`. Check inside:

```bash
leviathan4@leviathan:~$ ls -la .trash/
```

Output:

```
-r-sr-x--- 1 leviathan5 leviathan4 14944 Apr  3 15:19 bin
```

A setuid binary called `bin`, owned by `leviathan5`. Same structure as every previous level.

---

### Step 3 — Run the Binary

```bash
leviathan4@leviathan:~$ .trash/bin
```

Output:

```
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010
```

No prompt, no argument, it runs and immediately prints a string of space-separated groups of `0`s and `1`s. This is binary-encoded output. The binary is reading the `leviathan5` password file and printing its contents encoded in binary (base-2).

---

### Understand Binary and ASCII Encoding

> Every character a computer stores is ultimately a number. The **ASCII** standard defines the mapping: the letter `A` is 65, `B` is 66, `a` is 97, and so on. **Binary** is simply another way to write those same numbers, base 2 instead of base 10. The letter `T` is decimal 84, which in 8-bit binary is `01010100`. Each space-separated group in the output is one character of the password.

To reverse the encoding manually:

1. Take a group, e.g. `01010100`
2. Convert from binary to decimal: `64 + 16 + 4 = 84`
3. Look up decimal 84 in an ASCII table: `T`

With eleven groups in the output, doing this by hand is tedious but possible. A one-liner is faster.

|Representation|Value for the letter `T`|
|---|---|
|Binary|`01010100`|
|Decimal|`84`|
|ASCII|`T`|

---

### Step 4 — Decode the Binary Output

First, strip the spaces from the output so the bit string is continuous, then pipe it through Perl's `pack` function:

```bash
leviathan4@leviathan:~/.trash$ echo 010101000110100101110100011010000011010001100011011011110110101101100101011010010000 1010 | perl -lpe '$_=pack"B*",$_'
```

Breaking down the command:

|Component|Purpose|
|---|---|
|`echo <bitstring>`|Feeds the raw binary string to standard input|
|`perl -lpe '...'`|Runs a Perl one-liner: `-l` handles newlines, `-p` prints each line, `-e` specifies the code|
|`pack "B*", $_`|Interprets the input as a binary bit string and converts it to the corresponding ASCII characters|

Output:

```
<password for leviathan5>
```

That is the password for [Leviathan Level 5 → 6](leviathan-level-5-level-6.md).

---

## Key Takeaways

- **Not every Leviathan level requires `ltrace`.** The binary here reveals its own output without any tracing, it prints the encoded password directly to the terminal. Recognising the output format (binary encoding) is the entire puzzle. Reaching for `ltrace` reflexively is still reasonable, but learning to read output first saves time.
- **Binary-to-ASCII is a fundamental encoding, not obfuscation.** The binary representation of a character is just its ASCII value written in base 2. Knowing the ASCII table (or knowing where to look it up) and understanding base conversion is sufficient to decode it by hand.
- **Hidden directories (`.trash`) follow the same convention as hidden files.** A leading `.` hides the entry from plain `ls`. The `-a` flag surfaces it. This is consistent across all Leviathan levels, `ls -la` on arrival is never wasted.

---