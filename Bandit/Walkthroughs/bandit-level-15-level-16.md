> **Level Goal:** The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.
> 
> **Helpful note:** Getting "DONE", "RENEGOTIATING" or "KEYUPDATE"? Read the "CONNECTED COMMANDS" section in the manpage.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit15`                                                           |
| Password | *found in [Bandit Level 14 → Level 15](bandit-level-14-level-15.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `openssl s_client` — Open an **SSL/TLS connection** to a server
- `cat` — Print file contents to the terminal
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [Secure Socket Layer/Transport Layer Security on Wikipedia](https://en.wikipedia.org/wiki/Transport_Layer_Security)
- [OpenSSL Cookbook - Testing with OpenSSL](https://www.feistyduck.com/library/openssl-cookbook/online/testing-with-openssl/index.html)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit15@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 14 → Level 15](bandit-level-14-level-15.md).

---

### Understand the Problem

The previous level used `nc` to send data over a plain TCP connection. This level is identical in structure, connect to a port, send the password, receive the next one, except the connection must be **encrypted with SSL/TLS**.

Plain `nc` cannot do this. We need a tool that speaks SSL/TLS. That tool is `openssl s_client`.

**What is SSL/TLS?** [SSL/TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) is the **encryption layer** used by HTTPS, email, and most secure internet protocols. Before any data is exchanged, the client and server perform a **handshake**: they agree on an **encryption method and exchange keys**. After that, all data sent in either direction is encrypted. The service on port 30001 requires this **handshake** before it will accept input.

---

### Step 2 — Consult `man openssl` and the `s_client` Subcommand

`openssl` is a toolkit with many subcommands. The one we want is `s_client`:

```bash
bandit15@bandit:~$ man openssl-s_client
```

The relevant excerpt:

```
openssl-s_client - SSL/TLS client program

The s_client command implements a generic SSL/TLS client which
connects to a remote host using SSL/TLS. It is a useful diagnostic
tool for SSL servers.

-connect host:port
        This specifies the host and optional port to connect to.

-ign_eof
        Inhibit shutting down the connection when end of file is
        reached in the input. By default when the input hits EOF,
        the connection is closed down.
```

The `-ign_eof` flag is the solution to the hint in the level goal. Without it, typing the password and pressing `Enter` may trigger an EOF signal that closes the connection before the server can respond. With `-ign_eof`, the connection stays open after your input ends and you can read the server's reply.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 3 — Connect and Submit the Password

```bash
bandit15@bandit:~$ openssl s_client -connect localhost:30001 -ign_eof
```

`openssl s_client` will print a large block of SSL handshake information — certificate details, cipher suite, protocol version. This is normal. Scroll past it until you see:

```
...

---
read R BLOCK
```

That means the connection is established and the server is waiting for input. Type or paste the current `bandit15` password and press `Enter`:

```
<current_password>
Correct!
<next_password>
```

Here is what each part does:

|Part|What it does|
|---|---|
|`openssl s_client`|Opens an SSL/TLS encrypted connection to a server|
|`-connect localhost:30001`|Connects to port 30001 on the local machine|
|`-ign_eof`|Keeps the connection open after your input ends so you can read the reply|

That is the password for [Bandit Level 16 → Level 17](bandit-level-16-level-17.md).

---

> **Why Not Just Use `nc`?**
> 
> The command`nc localhost 30001` would open a TCP connection, but the service on port 30001 expects a TLS handshake immediately. If it does not receive one, it will either close the connection or send back garbled data. 
> `nc` has no knowledge of TLS — it sends and receives raw bytes only. `openssl s_client` handles the handshake transparently, so by the time you see the `read R BLOCK` prompt, encryption is already in place and the server is ready to accept your password as if it were a plain connection.

---

### Step 4 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit15@bandit:~$ exit
```

---

## Key Takeaways

- SSL/TLS encrypts a connection before any application data is exchanged. A plain TCP tool like `nc` cannot connect to a TLS-only service.
- `openssl s_client -connect host:port` is the standard diagnostic tool for testing SSL/TLS services — it handles the handshake and then passes your keyboard input through the encrypted connection.
- The handshake output printed by `s_client` is informational. The connection is ready when you see `read R BLOCK`.
- `-ign_eof` prevents the connection from closing when your input ends. Without it, pressing `Enter` after the password can shut the connection down before the server replies.
- "DONE", "RENEGOTIATING", or "KEYUPDATE" in the output means you sent a character that `s_client` interpreted as a command rather than data. The fix is `-ign_eof`.

---
