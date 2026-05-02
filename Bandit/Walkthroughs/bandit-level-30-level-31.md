> **Level Goal:** There is a git repository at `ssh://bandit30-git@bandit.labs.overthewire.org/home/bandit30-git/repo` via port `2220`. The password for `bandit30-git` is the same as for `bandit30`.
> 
> Clone the repository and find the password for the next level.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit30`                                                           |
| Password | *found in [Bandit Level 29 → Level 30](bandit-level-29-level-30.md)* |

---

## Commands Used

- `git clone` — Copy a remote repository to your local machine
- `git log` — Show the commit history
- `git branch -a` — List all branches including remote-tracking
- `git tag` — List tags
- `git show` — Show the contents of a tag or commit
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
$ git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
$ cd repo
```

Enter the `bandit30` password when prompted.

---

### Step 2 — Check the Obvious Places First

```bash
$ cat README.md
just an empty file... muahaha
```

```bash
$ git log --oneline
bd393e0 (HEAD -> master, origin/master, origin/HEAD) initial commit of README.md
```

One commit, nothing useful. Check branches:

```bash
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

Only one branch. Commits and branches are dead ends. The password is hidden somewhere else.

---

### Understand git Tags

A **tag** is a named reference to a specific point in git history — most commonly used to mark release versions (e.g. `v1.0.0`). Unlike branches, tags do not move as new commits are added; they are fixed labels.

There are two kinds of tags:

|Type|What it is|
|---|---|
|**Lightweight tag**|A plain pointer to a commit — just a name and a hash, nothing else.|
|**Annotated tag**|A full git object with its own message, author, date, and optionally a GPG signature. Can store arbitrary data in the message.|

Tags are not transferred by `git clone` automatically in the same way branches are — but `git clone` does fetch them. They are simply easy to overlook because they do not show up in `git log` or `git branch -a`.

|Command|What it does|
|---|---|
|`git tag`|Lists all tags in the repository.|
|`git show <tagname>`|Shows the tag's message and the commit it points to. For annotated tags, the message can contain arbitrary content.|

---

### Step 3 — List Tags

```bash
$ git tag
secret
```

One tag: `secret`.

---

### Step 4 — Show the Tag

```bash
$ git show secret
```

For an annotated tag this prints the tag message before the commit details. The output will look like:

```
<password>
```

The password is stored directly in the tag's message.

That is the password for [Bandit Level 31 → Level 32](bandit-level-31-level-32.md).

---

## Key Takeaways

- **`git tag`** is the third place to check in a repository after commits and branches. Tags are easy to overlook — they appear in neither `git log` nor `git branch -a`.
- **Annotated tags carry a message** that can contain arbitrary text. They are git objects in their own right, not just pointers. `git show <tagname>` reveals that message.
- The search order for secrets in a git repository is now complete across these four levels: **working tree files → commit history → branches → tags**. A stash (`git stash list`) rounds out the list but was not needed here.
- Tags are commonly used in real repositories to mark releases. An annotated tag on a release commit is a plausible place for a developer to leave a note — including one they should not have.

---
