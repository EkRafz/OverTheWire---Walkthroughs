> **Level Goal:** The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit14`                                                           |
| Password | *found in [Bandit Level 13 → Level 14](bandit-level-13-level-14.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `cat` — Print file contents to the terminal
- `nc` — **Read** and **write** data across network connections
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [How the Internet works in 5 minutes (YouTube)](https://www.youtube.com/watch?v=7_LPdttKXPc)
- [IP Addresses](https://computer.howstuffworks.com/web-server5.htm)
- [IP Address on Wikipedia](https://en.wikipedia.org/wiki/IP_address)
- [Localhost on Wikipedia](https://en.wikipedia.org/wiki/Localhost)
- [Ports](https://computer.howstuffworks.com/web-server8.htm)
- [Port (computer networking) on Wikipedia](https://en.wikipedia.org/wiki/Port_\(computer_networking\))

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit14@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 13 → Level 14](bandit-level-13-level-14.md).

---

### Understand the Problem

This level introduces a new concept: **network sockets**. Instead of reading a file, we need to connect to a service running on this machine, **send** it the **current password**, and it will respond with the next one.

Two terms to understand:

- **localhost** — the hostname that always refers to the machine you are currently on. Connecting to `localhost` never leaves the server — the traffic stays internal.
- **Port** — a number (0–65535) that **identifies a specific service** on a machine. A single machine can run **many services simultaneously**, each on its own port. SSH uses port 22 by default; this level's service listens on port 30000.

The tool for this is `nc` (netcat) — sometimes called the "Swiss army knife" of networking. It **opens a raw TCP connection** to any host and port, then lets you **send** and **receive data** through it.

---

### Step 2 — Get the Current Password

We need `bandit14`'s password to send to the service. We retrieved it at the end of the previous level, but it is also readable directly from the password file now that we are logged in as `bandit14`:

```bash
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
<current_password>
```

---

### Step 3 — Consult `man nc`

```bash
bandit14@bandit:~$ man nc
```

The relevant excerpt:

```
nc [options] host port

nc (netcat) is a simple utility which reads and writes data
across network connections using the TCP or UDP protocol.

It is designed to be a reliable back-end tool that can be used
directly or easily driven by other programs and scripts.
```

With no flags, `nc host port` opens a **TCP connection** to the given host and port, then connects your keyboard input to the socket and prints anything the server sends back. That is all we need here.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 4 — Connect and Submit the Password

```bash
bandit14@bandit:~$ nc localhost 30000
```

The connection opens and waits for input. Type or paste the current `bandit14` password and press `Enter`:

```
<current_password>
Correct!
<next_password>
```

Here is what each part does:

|Part|What it does|
|---|---|
|`nc`|Opens a raw TCP connection|
|`localhost`|Connects to the service running on this same machine|
|`30000`|The port the service is listening on|

The service reads the line you sent, checks it against the known password for `bandit14`, and responds with the password for the next level. That is the password for [Bandit Level 15 → Level 16](bandit-level-15-level-16.md).

---

### Step 5 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit14@bandit:~$ exit
```

---

## Key Takeaways

- **localhost** always refers to the **current machine**. Traffic sent to `localhost` never leaves the server.
- A **port** identifies a specific service on a machine. Many services can run simultaneously, each on a different port.
- `nc host port` **opens** a raw **TCP connection** and lets you **send** and **receive data** interactively — no protocol overhead, no framing, just bytes.
- This is the foundation of how almost all internet communication works: a client connects to a host and port, sends data, and reads the response. HTTP, SSH, and SMTP all follow this same basic model.

---
