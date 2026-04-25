> **Level Goal:** Visit the [Level 3](https://overthewire.org/wargames/bandit/bandit3.html) page to find out what you need for this level.

---

## Connection Details

| Field    | Value                                                         |
| -------- | ------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                 |
| Port     | `2220`                                                        |
| Username | `bandit2`                                                     |
| Password | *found in [Bandit Level 1 → Level 2](bandit-level-1-level-2.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls` — List directory contents
- `cat` — Print file contents to the terminal

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 1 → Level 2](bandit-level-1-level-2.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit2@bandit:~$ ls
spaces in this filename
```

There is one file. Its name is `spaces in this filename` — and that's exactly the problem. The shell uses spaces to separate arguments, so passing this filename directly will confuse it.

---

### Step 3 — The Problem with Spaces in Filenames

Your first instinct might be to run:

```bash
bandit2@bandit:~$ cat spaces in this filename
```

The shell doesn't see one filename — it sees **four separate arguments**: `spaces`, `in`, `this`, and `filename`. It will try to open four different files and fail on all of them.

---

### Step 4 — Read the File Correctly

There are two standard ways to handle filenames with spaces.

**Option A — Escape the spaces with `\`** _(used here)_

Put a **backslash** (`\`) before each space. This tells the shell to treat the space as part of the filename rather than as an argument separator:

```bash
bandit2@bandit:~$ cat ./spaces\ in\ this\ filename
```

The `./` prefix is the relative path to the current directory, which avoids any ambiguity about where the file is. This was covered in [Bandit Level 1 → Level 2](bandit-level-1-level-2.md).

**Option B — Wrap the filename in quotes**

Single or double quotes tell the shell to treat everything inside as one single argument, spaces included:

```bash
bandit2@bandit:~$ cat "./spaces in this filename"
```

Both options produce the same result. Quoting is often faster to type when a filename has many spaces.

The file contains the password for [Bandit Level 3 → Level 4](bandit-level-3-level-4.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit2@bandit:~$ exit
```

---

## Key Takeaways

- The shell splits input on spaces, so a filename with spaces must be handled explicitly — either by escaping each space with `\` or by wrapping the whole name in quotes.
- Both `./name\ with\ spaces` and `"./name with spaces"` are valid — quoting is usually faster when there are many spaces.

---
