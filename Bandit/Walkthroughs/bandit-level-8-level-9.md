> **Level Goal:** The password for the next level is stored in the file `data.txt` and is the only line of text that occurs only once.

---

## Connection Details

| Field    | Value                                                            |
| -------- | ---------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                    |
| Port     | `2220`                                                           |
| Username | `bandit8`                                                        |
| Password | *found in [Bandit Level 7 → Level 8](bandit-level-7-level-8.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `wc` — Count lines, words, or bytes in a file
- `sort` — Sort lines of text
- `uniq` — Filter adjacent duplicate lines
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see  [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 7 → Level 8](bandit-level-7-level-8.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit8@bandit:~$ ls
data.txt
```

One file: `data.txt`. Check its size before doing anything else:

```bash
bandit8@bandit:~$ wc -l data.txt
1001 data.txt
```

1001 lines. Most of them are duplicates — we need to find the one line that appears exactly once.

---

### Step 3 — Understand the Problem

The file contains **1001 lines**, the vast majority of which are repeated. We need the single unique line. Two tools handle this job together:

- `sort` — rearranges lines into alphabetical order
- `uniq` — compares **adjacent** lines and filters duplicates

The critical constraint is that `uniq` only **detects duplicates** when they are **next to each other**. A file with duplicates scattered throughout it will fool `uniq` into thinking everything is unique. **Sorting** first **guarantees that all identical lines are grouped together**, so `uniq` can do its job correctly.

---
### Why Sort First?

This is worth understanding clearly. Consider a simplified example:

```
apple
banana
apple
cherry
banana
```

Running `uniq -u` on this directly would print all five lines — none are adjacent duplicates. After `sort`:

```
apple
apple
banana
banana
cherry
```

Now `uniq -u` sees the duplicates grouped and correctly outputs only `cherry`. The same logic applies to `data.txt` at 1001 lines.

---

### Step 4 — Consult `man sort` and `man uniq`

```bash
bandit8@bandit:~$ man sort
```

The relevant excerpt:

```
sort [OPTION]... [FILE]...

Write sorted concatenation of all FILE(s) to standard output.
```

With no flags, `sort` sorts lines alphabetically. That's all we need here.

```bash
bandit8@bandit:~$ man uniq
```

The relevant excerpt:

```
uniq [OPTION]... [INPUT [OUTPUT]]

Filter adjacent matching lines from INPUT (or standard input),
writing to OUTPUT (or standard output).

-u, --unique
        only print unique lines
```

So `-u` tells `uniq` to print only lines that are **not** repeated — exactly what the level asks for.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 5 — Run the Command

```bash
bandit8@bandit:~$ sort data.txt | uniq -u
<password>
```

Here is what each part does:

| Part            | What it does                                                                                    |
| --------------- | ----------------------------------------------------------------------------------------------- |
| `sort data.txt` | Reads `data.txt` and outputs all lines sorted alphabetically, grouping identical lines together |
| \|              | Pipes `sort`'s output directly into `uniq` as its input                                         |
| `uniq -u`       | Reads the sorted stream and prints only lines that appear exactly once                          |

One line is printed. That is the password for [bandit level 9 → level 10](bandit-level-9-level-10.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit8@bandit:~$ exit
```

---

## Key Takeaways

- `uniq` only detects duplicates that are **adjacent**. Always `sort` first when duplicates may be scattered throughout a file.
- `uniq -u` prints only lines that occur exactly once — lines that appear two or more times are suppressed entirely.
- `sort FILE | uniq -u` is a standard Unix pattern for finding unique lines in a file with many duplicates.
- The pipe `|` connects two commands: the output of `sort` becomes the input of `uniq` without writing a temporary file.

---