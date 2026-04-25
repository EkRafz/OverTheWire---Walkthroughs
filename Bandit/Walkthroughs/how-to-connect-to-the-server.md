> **Level Goal:** Log into the game server using SSH. Once logged in, visit the [Level 0](https://overthewire.org/wargames/bandit/bandit0.html) page to find out how to beat the next level.

---

## Connection Details

| Field | Value                         |
| ----- | ----------------------------- |
| Host  | `bandit.labs.overthewire.org` |
| Port  | `2220`                        |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `man` — View the manual for any command
- `ls` — List directory contents
- `cat` — Print file contents to the terminal

> **Reference:** [SSH man page](https://manpages.ubuntu.com/manpages/noble/man1/ssh.1.html) · [SSH on Wikipedia](https://en.wikipedia.org/wiki/Secure_Shell) · [SSH with non-standard ports](https://itsfoss.com/ssh-to-port/)

---

## Solution

### Step 1 — Understand the SSH Command

The level hint tells us to use `ssh`. If you're unsure how it works, consult the manual:

```bash
man ssh
```

The relevant excerpt:

```
ssh connects and logs into the specified destination, which may be
specified as either [user@]hostname or a URI of the form
ssh://[user@]hostname[:port].

-p port
    Port to connect to on the remote host.
```

---

### Step 2 — Connect to the Server

There are two equivalent ways to connect:

**Option A — URI format**

```bash
ssh ssh://bandit0@bandit.labs.overthewire.org:2220
```

Breaking down the URI `ssh://[userinfo]@[host]:[port]`:

| Component | Value                         |
| --------- | ----------------------------- |
| Scheme    | `ssh`                         |
| User      | `bandit0`                     |
| Host      | `bandit.labs.overthewire.org` |
| Port      | `2220`                        |

**Option B — Flag format** _(more common)_

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

The `-p` flag specifies the port. The `user@host` syntax bundles username and hostname together.

---

### Step 3 — Accept the Host Fingerprint

On first connection you'll see:

```
Are you sure you want to continue connecting (yes/no/fingerprint)?
```

Type `yes`. This adds the host to `~/.ssh/known_hosts` so the prompt won't appear again.

---

### Step 4 — Enter the Password

You'll reach the login prompt:

```
bandit0@bandit.labs.overthewire.org's password:
```

Enter the level password. The password won't be visible as you type — that's normal.

---

### Step 5 — Exit

To exit the SSH session:

```bash
bandit0@bandit:~$ exit
```

---

## Key Takeaways

- `ssh` supports both URI syntax and `user@host -p port` syntax — both are valid.
- `man <command>` is your first stop when you don't know how a tool works.
- First-time SSH connections prompt you to verify the host fingerprint — this is expected behaviour.

---
