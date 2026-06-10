> **Level Goal:** There is no information for this level, intentionally. Figure it out yourself.

---

## Connection Details

| Field    | Value                            |
| -------- | -------------------------------- |
| Host     | `leviathan.labs.overthewire.org` |
| Port     | `2223`                           |
| Username | `leviathan0`                     |
| Password | `leviathan0`                     |

---

## Commands Used

- `ls -la` — List all files including hidden, with permissions and ownership
- `grep` — Search for lines matching a pattern inside a file

> **Reference:** [grep man page](https://man7.org/linux/man-pages/man1/grep.1.html)

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh leviathan0@leviathan.labs.overthewire.org -p 2223
```

---

### Step 2 — Look for Hidden Files

In [Level 0](leviathan-level-0.md), `ls -la` showed a hidden directory called `.backup`. Inspect it:

```bash
leviathan0@leviathan:~$ ls -la .backup
```

Output:

```
-rw-r----- 1 leviathan1 leviathan0 2346 Jan  1 00:00 bookmarks.html
```

A browser bookmark export, plain HTML, potentially hundreds of entries.

---

### Understand Hidden Files and Directories

On a Unix system, any file or directory whose name begins with a `.` is **hidden** — it won't appear in a plain `ls` listing. This is a convention, not a security control; the files are fully accessible to anyone with the right permissions.

> The `-a` flag reveals hidden entries. The `-l` flag adds the long format: permissions, ownership, size, and modification time. The `.backup` directory is owned by `leviathan1` but has group read permission for `leviathan0`, that combination is the signal it's meant to be found.

|Command|What it shows|
|---|---|
|`ls`|Visible files and directories only|
|`ls -a`|All files including hidden (`.` prefix)|
|`ls -la`|All files, long format — permissions and dates|

---

### Understand `grep`

Reading through hundreds of bookmark entries manually isn't practical. `grep` filters a file line by line and prints only the lines matching a pattern.

> The syntax is `grep <pattern> <file>`. The `-i` flag makes matching case-insensitive, catching `Password`, `PASSWORD`, and `password` equally. Since the file is a bookmark export, `password` is a reasonable first pattern to try.

---

### Step 3 — Extract the Password

```bash
leviathan0@leviathan:~$ grep -i password .backup/bookmarks.html
```

Output (abbreviated):

```html
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordfile" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">leviathan1 password: <password></A>
```

That is the password for [Leviathan Level 1 → 2](leviathan-level-1-level-2.md).

---

## Key Takeaways

- **`ls -la` on arrival is the standard first move.** The `-a` flag reveals hidden files and directories silently omitted by plain `ls`. Most Leviathan levels hide something in the home directory.
- **File ownership and permissions carry information.** The `.backup` directory being owned by `leviathan1` but group-readable by `leviathan0` is an intentional signal, not noise.
- **`grep` beats manual reading.** Any time you're looking for a specific string inside a large or unfamiliar file, `grep` is faster than opening it in a pager.
- **Sensitive data turns up in unexpected places.** Bookmark exports, config files, and shell history are all common sources of accidentally stored credentials, in CTFs and in production systems.

---