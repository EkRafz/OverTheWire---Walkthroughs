> **Level Goal:** The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don't. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.
> 
> **Helpful note:** Getting "DONE", "RENEGOTIATING" or "KEYUPDATE"? Read the "CONNECTED COMMANDS" section in the manpage.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit16`                                                           |
| Password | *found in [Bandit Level 15 → Level 16](bandit-level-15-level-16.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `nmap` — Scan for open ports and services
- `openssl s_client` — Open an SSL/TLS connection to a server
- `mktemp -d` — Create a temporary working directory
- `chmod` — Change file permissions
- `exit` — Close the SSH session

---

## Helpful Reading Material

- [Port scanner on Wikipedia](https://en.wikipedia.org/wiki/Port_scanner)

---

## Solution

### Step 1 — Connect to the Server

If you haven't used SSH before, see [How to connect to the server](how-to-connect-to-the-server.md) for a full explanation.

```bash
ssh bandit16@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 15 → Level 16](bandit-level-15-level-16.md).

---

### Step 2 — Understand the Problem

There are up to 1000 ports to check (31000–32000). We need to:

1. Find which ports in that range have a service listening
2. Find which of those speak SSL/TLS
3. Submit the password to each SSL/TLS port until one returns new credentials instead of echoing back what we sent

Checking each port by hand would take hours. This is exactly what a **port scanner** does automatically. The standard tool for this is `nmap`.

---

### Step 3 — Consult `man nmap`

```bash
bandit16@bandit:~$ man nmap
```

The relevant excerpt:

```
nmap [scan type] [options] {target}

Nmap ("Network Mapper") is an open source tool for network
exploration and security auditing.

-p port ranges
        Only scan specified ports. E.g. -p 31000-32000

-sV
        Probe open ports to determine service/version info.
        This also reveals whether a service is using SSL/TLS.
```

`-sV` is the key flag here — it does not just check whether a port is open, it probes each open port to identify what service is running and whether it uses SSL/TLS. That eliminates the need to test each port manually with `openssl`.

> **Tip — Navigating `man` pages:** Press `/` then type a keyword and hit `Enter` to search. Press `n` for the next match. Press `q` to quit.

---

### Step 4 — Scan the Port Range

```bash
bandit16@bandit:~$ nmap -sV -p31000-32000 localhost
```

Here is what each part does:

| Part            | What it does                                                            |
| --------------- | ----------------------------------------------------------------------- |
| `nmap`          | Runs the port scanner                                                   |
| `-sV`           | Probes each open port to detect the service and whether it uses SSL/TLS |
| `-p31000-32000` | Restricts the scan to ports in this range                               |
| `localhost`     | Scans the local machine                                                 |

The output will look something like this:

```
PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo    OpenSSL
31691/tcp open  echo
31790/tcp open  ssl/unknown OpenSSL
31960/tcp open  echo
```

Ports listed as `echo` simply reflect back whatever you send — those are the **decoys**. Ports listed with `ssl/` speak TLS. Of the SSL ports, one runs `ssl/echo` (still a decoy) and one runs `ssl/unknown` — that is the **target**.

---

### Step 5 — Submit the Password to the SSL/Unknown Port

Connect to the `ssl/unknown` port using `openssl s_client` with `-ign_eof` (see [Bandit Level 15 → Level 16](bandit-level-15-level-16.md) for a full explanation of these flags):

```bash
bandit16@bandit:~$ openssl s_client -connect localhost:31790 -ign_eof
```

Wait for the handshake output to finish, then type or paste the current `bandit16` password and press `Enter`:

```
<current_password>
Correct!
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

This level does not return a password — it returns a **private SSH key**. We need to save it to connect as `bandit17`.

---

### Step 6 — Save the Private Key

Create a temporary working directory and save the key there:

```bash
bandit16@bandit:~$ mktemp -d
/tmp/tmp.xK7Lm2pQnR
bandit16@bandit:~$ cd /tmp/tmp.xK7Lm2pQnR
```

Open a new file and paste the private key — everything from `-----BEGIN RSA PRIVATE KEY-----` through `-----END RSA PRIVATE KEY-----` — then save it:

```bash
bandit16@bandit:/tmp/tmp.xK7Lm2pQnR$ cat > sshkey.private
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

Press `Ctrl + D` to close the file once the key is pasted. Then restrict permissions so SSH will accept it:

```bash
bandit16@bandit:/tmp/tmp.xK7Lm2pQnR$ chmod 600 sshkey.private
```

---

### Step 7 — Log Out and Connect as bandit17

Exit the current session:

```bash
bandit16@bandit:/tmp/tmp.xK7Lm2pQnR$ exit
```

On your local machine, download the key with `scp`:

```bash
$ scp -P 2220 bandit16@bandit.labs.overthewire.org:/tmp/tmp.xK7Lm2pQnR/sshkey.private ./bandit17.key
$ chmod 600 bandit17.key
```

Then connect as `bandit17`:

```bash
$ ssh -i bandit17.key bandit17@bandit.labs.overthewire.org -p 2220
```

Once logged in as `bandit17`, read the password directly:

```bash
bandit17@bandit:~$ cat /etc/bandit_pass/bandit17
<password>
```

That is the password for [Bandit Level 17 → Level 18](bandit-level-17-level-18.md).

---

### Step 8 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit17@bandit:~$ exit
```

---

## Key Takeaways

- `nmap -sV -p range host` scans a port range and identifies services, including whether they use SSL/TLS. This is far faster than probing ports manually.
- An `echo` service **reflects back** whatever you send — a **common decoy pattern**. The **target** is always the port that **responds differently**.
- When a level returns a private SSH key instead of a password, save it to a file, `chmod 600` it, and use `ssh -i` to connect — the same pattern as [Bandit Level 13 → Level 14](bandit-level-13-level-14.md).
- `cat > filename` followed by `Ctrl + D` is a quick way to write multi-line content to a file without a text editor.
- Always `chmod 600` a private key before use. SSH will **refuse a key** that is **readable** by **other users**.

---
