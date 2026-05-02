> **Level Goal:** There is a git repository at `ssh://bandit28-git@bandit.labs.overthewire.org/home/bandit28-git/repo` via port `2220`. The password for `bandit28-git` is the same as for `bandit28`.
> 
> Clone the repository and find the password for the next level.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit28`                                                           |
| Password | *found in [Bandit Level 27 → Level 28](bandit-level-27-level-28.md)* |

---

## Commands Used

- `git clone` — Copy a remote repository to your local machine
- `git log` — Show the commit history
- `git show` — Show the contents of a specific commit
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
$ git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
$ cd repo
```

Enter the `bandit28` password when prompted.

---

### Step 2 — Read the Current File

```bash
$ cat README.md
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: xxxxxxxxxx
```

The password has been redacted. Someone committed the real password and then replaced it. Git stores the **complete history** of every change ever made — the original value is still in there.

---

### Understand git log and git show

**`git log`** lists every commit in the repository's history, from most recent to oldest. Each entry shows a commit hash, the author, the date, and the commit message.

**`git show <hash>`** displays the full details of a specific commit: the metadata and the **diff** — the exact lines that were added and removed. Lines prefixed with `+` were added; lines prefixed with `-` were removed.

A commit hash is the long hexadecimal string that uniquely identifies each commit. You only need to provide enough characters for it to be unambiguous — typically the first 7 are sufficient.

---

### Step 3 — Inspect the Commit History

```bash
$ git log
commit adc7f885a129... (HEAD -> master, origin/master)
Author: ...
Date:   ...

    fix info leak

commit a3437bddd447...
Author: ...
Date:   ...

    add missing data

commit cb630ec182b75...
Author: ...
Date:   ...

    initial commit
```

Three commits. The most recent one is titled `fix info leak` — that is almost certainly the commit that redacted the password. The real password will be in the commit just before it.

---

### Step 4 — Show the Redacting Commit

Pass the hash of the `fix info leak` commit to `git show`:

```bash
$ git show adc7f885a129baee883058b8a870739489f80194
```

The diff will look like this:

```diff
-  password: <bandit29_password>
+  password: xxxxxxxxxx
```

The line prefixed with `-` is what was **removed** by this commit — the real password before it was replaced with `xxxxxxxxxx`.

That is the password for [Bandit Level 29 → Level 30](bandit-level-29-level-30.md).

---

## Key Takeaways

- **Git never forgets.** Every version of every file that was ever committed is permanently stored in the repository history. Redacting sensitive data by making a new commit does not remove it — the original value remains visible in the diff of the redacting commit.
- **`git log`** is the entry point for history investigation. Read the commit messages — they often describe exactly what changed and point you directly to the relevant commit.
- **`git show <hash>`** shows the diff for a commit. Lines prefixed with `-` were removed; lines prefixed with `+` were added. Removed lines are exactly where redacted secrets live.
- The real-world implication is significant: if a secret is ever committed to a git repository — even briefly — it should be considered permanently compromised. The correct response is to rotate the credential, not just overwrite it in a follow-up commit.

---
