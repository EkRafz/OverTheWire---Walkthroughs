> **Level Goal:** There is a git repository at `ssh://bandit29-git@bandit.labs.overthewire.org/home/bandit29-git/repo` via port `2220`. The password for `bandit29-git` is the same as for `bandit29`.
> 
> Clone the repository and find the password for the next level.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit29`                                                           |
| Password | *found in [Bandit Level 28 → Level 29](bandit-level-28-level-29.md)* |

---

## Commands Used

- `git clone` — Copy a remote repository to your local machine
- `git log` — Show the commit history
- `git branch` — List branches
- `git checkout` — Switch to a different branch
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
$ git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
$ cd repo
```

Enter the `bandit29` password when prompted.

---

### Step 2 — Read the Current File

```bash
$ cat README.md
# Bandit Notes
Some notes for level30 of bandit.

## credentials

- username: bandit30
- password: <no passwords in production!>
```

The note says no passwords in production. That phrasing is a hint: the password may exist in a non-production branch.

---

### Step 3 — Check the Commit History

```bash
$ git log
```

Only a handful of commits, none with suspicious messages like the `fix info leak` from last level. The history on this branch is clean — the password was never here.

---

### Understand git Branches

A **branch** is an independent line of development within a repository. The default branch is typically called `main` or `master`. Repositories often have additional branches for features, testing, or development work that is kept separate from production.

`git clone` checks out the default branch but downloads references to all remote branches. Those branches exist on the remote and are visible locally as **remote-tracking branches** — they are not automatically checked out, but you can list and switch to them.

|Command|What it does|
|---|---|
|`git branch`|Lists local branches only.|
|`git branch -a`|Lists all branches: local and remote-tracking.|
|`git checkout <branch>`|Switches to a branch, creating a local copy of a remote branch if needed.|

---

### Step 4 — List All Branches

```bash
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/sploits-dev
```

Two non-default remote branches: `dev` and `sploits-dev`. Given the hint about production, `dev` is the obvious place to look.

---

### Step 5 — Switch to the dev Branch

```bash
$ git checkout dev
Branch 'dev' set up to track 'origin/dev'.
Switched to a new branch 'dev'
```

---

### Step 6 — Read the File

```bash
$ cat README.md
# Bandit Notes
Some notes for level30 of bandit.

## credentials

- username: bandit30
- password: <password>
```

The password is present on the `dev` branch.

That is the password for [Bandit Level 30 → Level 31](bandit-level-30-level-31.md).

---

## Key Takeaways

- **Branches are independent lines of development.** The default branch is not the only place to look — sensitive data is often committed to feature or development branches that were never intended to be public, but are still part of the cloned repository.
- **`git branch -a`** is the first command to run when the current branch yields nothing. The `-a` flag is essential — without it, remote-tracking branches are invisible.
- **`git checkout <branch>`** switches context entirely. Files, history, and `git log` output all reflect the checked-out branch, not the one you started on.
- The "no passwords in production" note is a real-world pattern: developers sometimes treat non-production branches as lower-security, committing credentials they would never put on `main`. Those branches are still in the repository and fully accessible to anyone who clones it.
- Combined with the previous level: git history (commits) and git structure (branches) are both worth searching. Later levels add tags and stashes to that list.

---
