> **Level Goal:** There are 2 files in the home directory: `passwords.old` and `passwords.new`. The password for the next level is in `passwords.new` and is the only line that has been changed between `passwords.old` and `passwords.new`.
> 
> **Note:** If you have solved this level and see `Byebye!` when trying to log into bandit18, this is related to the next level, bandit19.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit17`                                                           |
| Password | *found in [Bandit Level 16 → Level 17](bandit-level-16-level-17.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `diff` — Compare two files line by line
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit17@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 16 → Level 17](bandit-level-16-level-17.md).

---

### Step 2 — List the Home Directory

```bash
bandit17@bandit:~$ ls
passwords.new  passwords.old
```

Two files. Both contain many lines — comparing them manually would be impractical. We need `diff`.

---

### Step 3 — Consult `man diff`

```bash
bandit17@bandit:~$ man diff
```

The relevant excerpt:

```
diff [OPTION]... FILES

Compare FILES line by line.

Output format (no flags):
  < line   — line only in the first file
  > line   — line only in the second file
  ---      — separator between changed blocks
```

`diff file1 file2` **compares two files** and **prints** only the **lines that differ**. Lines prefixed with `<` come from the first file; lines prefixed with `>` come from the second. Lines that are identical in both files are not printed at all.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 4 — Run the Command

```bash
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< <old_line>
---
> <new_line>
```

Here is what each part of the output means:

|Output|Meaning|
|---|---|
|`42c42`|Line 42 in the first file was **c**hanged to line 42 in the second|
|`< old_line`|The line as it appears in `passwords.old`|
|`---`|Separator between the old and new versions|
|`> new_line`|The line as it appears in `passwords.new`|

The line prefixed with `>` is the changed line in `passwords.new`. That is the password for [Bandit Level 18 → Level 19](bandit-level-18-level-19.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit17@bandit:~$ exit
```

---

## Key Takeaways

- `diff file1 file2` **compares two files** and prints only the differing lines — identical lines are suppressed entirely.
- Lines prefixed with `<` come from the first file; lines prefixed with `>` come from the second.
- The change descriptor `42c42` means line 42 was changed. Other codes are `a` (added) and `d` (deleted).
- `diff` is the foundation of tools like `git diff` — any time you need to see what changed between two versions of a file, `diff` is the right tool.

---
