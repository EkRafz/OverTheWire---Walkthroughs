> **Level Goal:** Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not `/bin/bash`, but something else. Find out what it is, how it works and how to break out of it.
> 
> **Note:** If you're a Windows user and typically use Powershell to SSH into bandit: Powershell is known to cause issues with the intended solution to this level. Use Command Prompt instead.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit25`                                                           |
| Password | *found in [Bandit Level 24 → Level 25](bandit-level-24-level-25.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `cat` — Print file contents to the terminal
- `ls` — List directory contents
- `more` — A terminal pager (displays file contents one screen at a time)
- `vi` — A terminal text editor

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh bandit25@bandit.labs.overthewire.org -p 2220
```

---

### Step 2 — Find the SSH Key for bandit26

```bash
bandit25@bandit:~$ ls
bandit26.sshkey
```

There is an SSH private key for `bandit26` in the home directory. Logging in with it seems straightforward:

```bash
bandit25@bandit:~$ ssh -i bandit26.sshkey bandit26@localhost -p 2220
```

The connection opens briefly, prints some text, and immediately closes. You are kicked back to the `bandit25` prompt. The login worked — but something is terminating the session the moment it starts.

---

### Understand the Problem

When SSH logs you in, it starts your **login shell** — the program defined for your user in `/etc/passwd`. For most users that is `/bin/bash`. For `bandit26` it is something else.

Check `/etc/passwd`:

```bash
bandit25@bandit:~$ cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

The shell is `/usr/bin/showtext`. Read it:

```bash
bandit25@bandit:~$ cat /usr/bin/showtext
#!/bin/sh

export TERM=linux

exec more ~/text.txt
exit 0
```

Every time `bandit26` logs in, this script runs `more ~/text.txt` and then exits. `more` is a **pager** — it displays file contents one screen at a time and waits for input if the file is longer than the terminal. If the file fits on screen, `more` prints it and exits immediately, which is why the session closes so fast.

The escape route is inside `more` itself: if you can get `more` to pause and wait for input, you can use its built-in commands to break out into a real editor — and from there, into a shell.

---

### Step 3 — Shrink Your Terminal Window

`more` only enters interactive mode when the content **does not fit on screen**. You need to make your terminal window short enough that `text.txt` cannot be displayed all at once.

**Resize your terminal window to be very short — around 5 lines tall.**

The exact number of lines does not matter as long as `more` is forced to pause. On most graphical terminal emulators, drag the bottom edge of the window upward.

---

### Step 4 — Log in Again

With the terminal window still very short, SSH in as `bandit26` again:

```bash
bandit25@bandit:~$ ssh -i bandit26.sshkey bandit26@localhost -p 2220
```

This time, `more` cannot fit `text.txt` on screen. It pauses and displays `--More--` at the bottom instead of exiting. **The session stays open.**

---

### Step 5 — Open vi from Inside more

`more` has an interactive command to open the current file in `vi`. While `more` is paused (showing `--More--`), press:

```
v
```

This drops you into `vi` editing `text.txt`. You are now inside a real text editor running as `bandit26`.

---

### Step 6 — Set a Shell and Escape from vi

`vi` can run shell commands and open new shells. The trick is to point `vi`'s shell setting at `/bin/bash` and then invoke it.

In `vi`, type the following commands (each starts with `:` to enter command mode):

```
:set shell=/bin/bash
```

Press `Enter`. Then:

```
:shell
```

Press `Enter`.

What each command does:

|Command|What it does|
|---|---|
|`:set shell=/bin/bash`|Overrides `vi`'s `shell` option, which controls what program `:shell` launches. Without this, it would inherit `/usr/bin/showtext` from the login shell — which would immediately exit again.|
|`:shell`|Launches the configured shell as a subprocess of `vi`, giving you an interactive bash prompt.|

You now have a working shell as `bandit26`.

---

### Step 7 — Read the Password

```bash
bandit26@bandit:~$ cat /etc/bandit_pass/bandit26
<password>
```

That is the password for bandit26. You will need it — and this shell — for [Bandit Level 26 → Level 27](bandit-level-26-level-27.md).

---

### Step 8 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit26@bandit:~$ exit
```

---

## Key Takeaways

- **`/etc/passwd` defines the login shell** for each user — not `/bin/bash` by default, just whatever path is in the last field of the user's record. Always check it when a session closes unexpectedly on login.
- **`more` is a pager**, not just a viewer. It has interactive commands (`v` to open `vi`, `q` to quit, space to advance) that only become accessible when it is forced into interactive mode — which only happens when content exceeds the terminal height.
- **Terminal window size is an input**. Programs query it via `TIOCGWINSZ` and behave differently based on it. Shrinking the window is a legitimate way to change program behaviour, not a hack.
- **`vi` can spawn a shell** with `:shell`. The shell it spawns is controlled by the `shell` option, which can be overridden mid-session with `:set shell=/path/to/shell`. This is a well-known `vi` escape technique and appears in many real-world privilege escalation scenarios (see [GTFOBins: vi](https://gtfobins.github.io/gtfobins/vi/)).
- The escape chain here — **restricted shell → pager → editor → shell** — illustrates how layered restrictions can be bypassed one step at a time by finding interactive surface in each layer.

---
