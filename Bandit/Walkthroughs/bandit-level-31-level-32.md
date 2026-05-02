> **Level Goal:** There is a git repository at `ssh://bandit31-git@bandit.labs.overthewire.org/home/bandit31-git/repo` via port `2220`. The password for `bandit31-git` is the same as for `bandit31`.
> 
> Clone the repository and find the password for the next level.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit31`                                                           |
| Password | *found in [Bandit Level 30 → Level 31](bandit-level-30-level-31.md)* |

---

## Commands Used

- `git clone` — Copy a remote repository to your local machine
- `git add` — Stage a file for commit
- `git commit` — Record staged changes to the repository
- `git push` — Upload local commits to a remote repository
- `cat` — Print file contents to the terminal

---

## Helpful Reading Material

- [Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)

---

## Solution

### Step 1 — Clone the Repository

Run this on your **local machine**:

```bash
$ git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
$ cd repo
```

Enter the `bandit31` password when prompted.

---

### Step 2 — Read the README

```bash
$ cat README.md
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master
```

The previous git levels were read-only — clone and inspect. This one requires you to **write** to the remote: create a file with specific content and push it to the server. The server will respond with the password.

---

### Step 3 — Check .gitignore

Before creating the file, check whether anything in the repository would prevent it from being tracked:

```bash
$ cat .gitignore
*.txt
```

The repository's `.gitignore` tells git to ignore all `.txt` files. If you try to `git add key.txt` without addressing this, git will silently skip it and your push will contain nothing.

---

### Understand .gitignore and Force-Adding

A `.gitignore` file lists patterns for files git should not track. Any file matching a pattern is excluded from `git add` by default.

To add a file that matches a `.gitignore` pattern, pass the `-f` (force) flag to `git add`:

```bash
git add -f <filename>
```

This overrides the ignore rule for that specific file. It does not modify `.gitignore` — it just tells git to track this file despite the rule.

---

### Step 4 — Create the File

```bash
$ echo 'May I come in?' > key.txt
```

Verify the content is exact — the server will check it:

```bash
$ cat key.txt
May I come in?
```

---

### Step 5 — Stage the File

Use `-f` to force-add past the `.gitignore` rule:

```bash
$ git add -f key.txt
```

Confirm it is staged:

```bash
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   key.txt
```

---

### Step 6 — Commit

```bash
$ git commit -m "add key.txt"
[master xxxxxxx] add key.txt
 1 file changed, 1 insertion(+)
 create mode 100644 key.txt
```

---

### Step 7 — Push to the Remote

```bash
$ git push origin master
```

Enter the `bandit31` password when prompted. The remote server receives the push, checks the file content, and responds with the password in the push output:

```
...
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote: 
remote: Well done! Here is the password for the next level:
remote: <password> 
remote: 
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
...
```

That is the password for [Bandit Level 32 → Level 33](bandit-level-32-level-33.md).

---

## Key Takeaways

- **`git push`** sends local commits to a remote repository. The basic form is `git push <remote> <branch>` — `origin` is the default name for the remote you cloned from.
- **`.gitignore`** patterns silently prevent files from being staged. If `git add` appears to do nothing, check `.gitignore` first — a matching pattern is the most common cause.
- **`git add -f`** force-adds a file that would otherwise be excluded by `.gitignore`. Use it deliberately — the ignore rule usually exists for a reason.
- The git workflow introduced across levels 27–31 in full: **clone → inspect → modify → stage → commit → push**. Most day-to-day git usage follows exactly this sequence.
- Servers can execute logic on push via **git hooks** — scripts that run automatically when a push is received. Here the server-side hook checked the file content and returned the password. In real repositories, hooks are used for CI triggers, linting, access control, and deployment.

---
