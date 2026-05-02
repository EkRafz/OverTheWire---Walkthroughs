> **Level Goal:** There is a setuid binary in the home directory that does the following: it makes a connection to localhost on the port you specify as a command line argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).
> 
> **Note:** Try connecting to your own network daemon to see if it works as you think.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit20`                                                           |
| Password | *found in [Bandit Level 19 → Level 20](bandit-level-19-level-20.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `ls -l` — List directory contents with permissions
- `nc` — Set up a listening network service
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit20@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 19 → Level 20](bandit-level-19-level-20.md).

---

### Step 2 — List the Home Directory

```bash
bandit20@bandit:~$ ls -l
-rwsr-x--- 1 bandit21 bandit20 15604 ... suconnect
```

One setuid binary: `suconnect`. It is owned by `bandit21` with the setuid bit set, so it runs with `bandit21`'s permissions. **Run it without arguments** to see its usage:

```bash
bandit20@bandit:~$ ./suconnect
Usage: ./suconnect <portnumber>
This program will make a connection to the given port on localhost using TCP.
If it receives the correct password from the connection, the next password is transmitted back.
```

---

### Understand the Problem

This level requires two things to happen simultaneously:

1. A **listener** — a service sitting on a port, ready to send the `bandit20` password to whatever connects to it
2. The **suconnect binary** — which connects to that port, reads the password, and if correct, sends back the `bandit21` password

Both need to be running at the same time on the same machine. We only have one terminal. The solution is **job control**: run the listener in the background with `&`, then run `suconnect` in the foreground.

---

### Understand `nc -l` and Job Control

`nc` can operate in two modes: client (connect to a port) or server (listen on a port). The `-l` flag puts it in listening mode.

```bash
bandit20@bandit:~$ man nc
```

The relevant excerpt:

```
-l      Used to specify that nc should listen for an incoming
        connection rather than initiate a connection to a remote host.

-p port
        Specifies the source port nc should use.
```

For job control, appending `&` to any command runs it in the background, freeing the terminal immediately. The shell prints a job number and process ID:

```
[1] 12345
```

The background job keeps running while you use the terminal normally.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 3 — Start the Listener in the Background

Pick any unused port in a high range (e.g. `59999`) and start `nc` listening on it. Pipe the current password into `nc` so it sends it automatically to whatever connects:

```bash
bandit20@bandit:~$ echo "<bandit20_password>" | nc -l -p 59999 &
[1] 12345
```

Here is what each part does:

| Part                | What it does                                                                |
| ------------------- | --------------------------------------------------------------------------- |
| `echo "<password>"` | Produces the password as a single line of text                              |
| \|                  | Pipes it into `nc` as the data to send when a client connects               |
| `nc -l -p 59999`    | Starts a TCP listener on port 59999 that sends the piped data on connection |
| `&`                 | Runs the whole command in the background                                    |

---

### Step 4 — Run suconnect

With the listener running in the background, connect to it using `suconnect`:

```bash
bandit20@bandit:~$ ./suconnect 59999
Read: <bandit20_password>
Password matches, sending next password
<bandit21_password>
[1]+ Done    echo "<bandit20_password>" | nc -l -p 59999
```

`suconnect` connects to port 59999, reads the password sent by `nc`, checks it against the stored `bandit20` password, and transmits the `bandit21` password back — which appears in your terminal. The background job then exits cleanly.

That is the password for [Bandit Level 21 → Level 22](bandit-level-21-level-22.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit20@bandit:~$ exit
```

---

## Key Takeaways

- `nc -l -p port` turns `nc` into a TCP listener. It waits for one connection, sends any piped input, and exits.
- Appending `&` to a command runs it as a background job, freeing the terminal immediately. Use `jobs` to list background jobs and `fg` to bring one back to the foreground.
- `echo "data" | nc -l -p port` is a concise way to serve a single line of text to the first client that connects.
- Some problems require **two processes running simultaneously**. Job control (`&`, `fg`, `bg`, `jobs`) is the single-terminal solution — an alternative is opening a second SSH session.

---
