> **Level Goal:** The password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user `bandit14`. For this level, you don't get the next password directly — instead you get a private SSH key that can be used to log into the next level.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit13`                                                           |
| Password | *found in [Bandit Level 12 → Level 13](bandit-level-12-level-13.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `scp` — Copy files securely over SSH
- `ls` — List directory contents
- `cat` — Print file contents to the terminal
- `chmod` — Change file permissions
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit13@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 12 → Level 13](bandit-level-12-level-13.md).

---

### Step 2 — List the Home Directory

Once logged in, run `ls` to see what's there:

```bash
bandit13@bandit:~$ ls
HINT  sshkey.private
```

No `data.txt` this time. Read the hint before doing anything else:

```bash
bandit13@bandit:~$ cat HINT
If you have trouble with this level, note the following:
1) As for all other levels, this level has a website with information:
   https://overthewire.org/wargames/bandit/bandit14.html
2) No, the level is not broken. To verify, see:
   https://status.overthewire.org/
3) The current version of OverTheWire prevents logging in from one
   level to another via localhost. Log out, and see 1)
4) If you get errors, read the error message on your screen.
   We mean it!
```

The critical point is hint 3: connecting to `bandit14@localhost` from within the server is **blocked**. Instead, we copy the private key to our local machine and connect directly from there as `bandit14`.

---

### Understand the Problem

Every time you have connected to a Bandit level, SSH has authenticated you with a password. SSH supports a second authentication method: **key pairs**.

A key pair consists of two files:

- A **private key** — kept secret, never shared, stays on your machine
- A **public key** — placed on the remote server in `~/.ssh/authorized_keys`

When you connect, SSH proves you hold the private key without ever sending it over the network. The server checks it against the stored public key and grants access if they match.

Here, the server already has `bandit14`'s public key. We have been given the matching private key. We exit, copy the key to our local machine with `scp`, and then pass it to `ssh` with the `-i` flag to authenticate as `bandit14`.

---

### Step 3 — Consult `man ssh` and `man scp`

```bash
bandit13@bandit:~$ man ssh
```

The relevant excerpt:

```
ssh [options] [user@]hostname

-i identity_file
        Selects a file from which the identity (private key) for
        public key authentication is read. The default is
        ~/.ssh/id_rsa. Identity files may also be specified on a
        per-host basis in the configuration file.
```

```bash
bandit13@bandit:~$ man scp
```

The relevant excerpt:

```
scp [options] source target

scp copies files between hosts on a network using SSH for
data transfer and authentication.

-P port
        Specifies the port to connect to on the remote host.
```

Note the capital `-P` for `scp` — unlike `ssh` which uses lowercase `-p`.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 4 — Copy the Key to Your Local Machine

Exit the current session:

```bash
bandit13@bandit:~$ exit
```

Back on your local machine, use `scp` to download the private key:

```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private ./bandit14.key
```

Here is what each part does:

|Part|What it does|
|---|---|
|`scp`|Copies files securely over SSH|
|`-P 2220`|Uses port 2220 (capital `-P`, unlike `ssh`'s lowercase)|
|`bandit13@bandit.labs.overthewire.org:~/sshkey.private`|Remote file to copy, in `user@host:path` format|
|`./bandit14.key`|Where to save it locally|

Enter the **`bandit13` password** when prompted. Then set the **correct permissions** — SSH **refuses** to use a private key **readable by anyone** other than you:

```bash
$ chmod 600 bandit14.key
```

---

### Step 5 — Log into bandit14 Using the Key

Connect directly to the server as `bandit14`:

```bash
$ ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
```

|Part|What it does|
|---|---|
|`-i bandit14.key`|Uses the private key for authentication instead of a password|
|`bandit14@bandit.labs.overthewire.org`|Connects as user `bandit14` directly to the Bandit server|
|`-p 2220`|Connects on port 2220|

SSH may ask you to confirm the host fingerprint — type `yes` and press `Enter`.

---

### Step 6 — Read the Password

Now logged in as `bandit14`, we can read the password file that was off-limits before:

```bash
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
<password>
```

This is the password for `bandit14`. Save it — it is required for [Bandit Level 14 → Level 15](bandit-level-14-level-15.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit14@bandit:~$ exit
```

---

## Key Takeaways

- SSH supports two authentication methods: **passwords** and **key pairs**. Key pairs are more secure and widely used in practice.
- A key pair has two parts: a **private key** (secret, never shared) and a **public key** (placed on the server). The private key proves identity without ever being transmitted.
- `scp -P port user@host:remote_path local_path` copies a file from a remote server to your local machine. Note the capital `-P` for port, unlike `ssh`.
- `chmod 600 keyfile` restricts a private key to owner-read/write only — SSH will refuse to use a key with **looser permissions**.
- `ssh -i keyfile user@host -p port` **authenticates** with a key file instead of a password.
- When `localhost` is **blocked**, **copy the key locally** and connect from your own machine instead.

---
