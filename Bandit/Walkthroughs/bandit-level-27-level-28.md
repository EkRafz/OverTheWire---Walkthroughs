> **Level Goal:** There is a git repository at `ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo` via port `2220`. The password for `bandit27-git` is the same as for `bandit27`.
> 
> Clone the repository and find the password for the next level.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit27`                                                           |
| Password | *found in [Bandit Level 26 → Level 27](bandit-level-26-level-27.md)* |

---

## Commands Used

- `git clone` — Copy a remote repository to your local machine
- `ls` — List directory contents
- `cat` — Print file contents to the terminal

---

## Helpful Reading Material

- [Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)

---

## Solution

### Understand the Problem

This level introduces **git** — a version control system that tracks changes to files over time. A **repository** (repo) is a directory whose history git manages. `git clone` copies a remote repository — including its full history — to your local machine.

From here on, several Bandit levels use git. Each one hides the password somewhere in the repository's structure: in a file, in a commit message, in a stashed change, or in a branch. This first git level is the simplest: the password is sitting in a file in the repository.

---

### Step 1 — Clone the Repository

Run this on your **local machine**, not on the OverTheWire server. The repository is served over SSH on a non-standard port, so the URL needs a port specification.

```bash
$ git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
```

Git will prompt for a password — enter the `bandit27` password you found in the previous level.

What each part of the URL means:

|Part|What it means|
|---|---|
|`ssh://`|Use SSH as the transport protocol.|
|`bandit27-git`|The username to authenticate as on the remote host.|
|`bandit.labs.overthewire.org`|The remote host.|
|`:2220`|The non-standard SSH port. Git SSH URLs include the port directly in the URL with this syntax.|
|`/home/bandit27-git/repo`|The path to the repository on the remote filesystem.|

Git clones the repository into a new directory called `repo` in your current working directory.

---

### Step 2 — Look Inside the Repository

```bash
$ cd repo
$ ls
README
```

One file: `README`.

---

### Step 3 — Read the File

```bash
$ cat README
The password to the next level is: <password>
```

That is the password for [Bandit Level 28 → Level 29](bandit-level-28-level-29.md).

---

## Key Takeaways

- **`git clone <url>`** copies a remote repository to your local machine, including the full commit history. The cloned directory is a complete, self-contained copy — not just the current files.
- **Git SSH URLs** use the format `ssh://user@host:port/path`. The port is part of the URL, not a separate flag, which differs from how plain `ssh` works (`ssh -p 2220 user@host`).
- This is the simplest possible git level — the password is in a file in the working tree. Later levels hide it in commit history, branches, stashes, and tags. The skill being introduced here is just: clone a repository and look at what's in it.
- Many real services (GitHub, GitLab, self-hosted servers) expose git over SSH. The clone URL pattern here is identical to what you use with those services every day.

---
