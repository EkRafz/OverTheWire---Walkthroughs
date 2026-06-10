> **Level Goal:** Log into the game server using SSH. Once logged in, find the password for the next level.

---

## Connection Details

|Field|Value|
|---|---|
|Host|`leviathan.labs.overthewire.org`|
|Port|`2223`|
|Username|`leviathan0`|
|Password|`leviathan0`|

---

## Commands Used

- `ssh` — Connect to remote machines securely

> **Reference:** [Leviathan on OverTheWire](https://overthewire.org/wargames/leviathan/) · [SSH man page](https://manpages.ubuntu.com/manpages/noble/man1/ssh.1.html)

---

## Solution

### Step 1 — Understand the SSH Command

The connection requires a non-standard port. Consult the manual if you're unfamiliar with the flag:

```bash
man ssh
```

The relevant excerpt:

```
-p port
    Port to connect to on the remote host.
```

---

### Step 2 — Connect to the Server

```bash
ssh leviathan0@leviathan.labs.overthewire.org -p 2223
```

Breaking down the command:

|Component|Value|
|---|---|
|`leviathan0@...`|Username and host combined|
|`-p 2223`|Non-standard port|

Enter the password `leviathan0` when prompted. The password won't be visible as you type — that's normal.

---

### Step 3 — Accept the Host Fingerprint

On first connection you'll see:

```
Are you sure you want to continue connecting (yes/no/fingerprint)?
```

Type `yes`. This adds the host to `~/.ssh/known_hosts` so the prompt won't appear again.

---

### Step 4 — Initial Recon

Once logged in, check what's in the home directory:

```bash
leviathan0@leviathan:~$ ls -la
```

You'll see a hidden directory called `.backup`. You have the foothold, finding what's inside it is the task for [Leviathan Level 0 → 1](leviathan-level-0-level-1.md).

---

## Key Takeaways

- **Leviathan uses port 2223**, not the default SSH port 22. The `-p` flag specifies a non-standard port; forgetting it produces a connection timeout, not an error message that tells you why.
- **`man <command>` is always your first stop** when a tool has an unfamiliar flag. The SSH man page is dense but `-p` is easy to find.
- **First-time SSH connections prompt for the host fingerprint.** This is expected behaviour — type `yes` to proceed.
- **`ls -la` on arrival is the standard first move.** The `-a` flag reveals hidden files and directories (those prefixed with `.`) that a plain `ls` silently omits.

---