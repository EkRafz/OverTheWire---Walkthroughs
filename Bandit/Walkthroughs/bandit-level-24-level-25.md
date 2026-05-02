> **Level Goal:** A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.
> 
> **Note:** You do not need to create new connections each time.

---

## Connection Details

| Field    | Value                                                                |
| -------- | -------------------------------------------------------------------- |
| Host     | `bandit.labs.overthewire.org`                                        |
| Port     | `2220`                                                               |
| Username | `bandit24`                                                           |
| Password | *found in [Bandit Level 23 → Level 24](bandit-level-23-level-24.md)* |

---

## Commands Used

- `ssh` — Connect to remote machines securely
- `nc` — Connect to a TCP port
- `nano` / `vim` — Text editors for writing the shell script
- `chmod` — Change file permissions
- `bash` — Run a shell script
- `sort` — Sort lines of text
- `uniq` — Filter duplicate lines
- `exit` — Close the SSH session

---

## Solution

### Step 1 — Connect to the Server

```bash
ssh bandit24@bandit.labs.overthewire.org -p 2220
```

Use the password you found at the end of [Bandit Level 23 → Level 24](bandit-level-23-level-24.md).

---

### Understand the Problem

The daemon on port 30002 expects a single line in this format:

```
<bandit24 password> <4-digit pin>
```

There are 10,000 possible pins (`0000` through `9999`). Trying them by hand is not feasible. The solution is to **generate all 10,000 candidate lines in a script**, pipe them all into a single `nc` connection, and look for the one response that is not a rejection.

The level note — _you do not need to create new connections each time_ — is the key hint. `nc` keeps the connection open as long as its stdin is open. If you pipe a file of 10,000 lines into `nc`, the daemon reads and responds to each one over the same connection. Opening 10,000 separate TCP connections would be slow and is unnecessary.

---

### Step 2 — Probe the Daemon

Before scripting anything, connect manually to understand the response format:

```bash
bandit24@bandit:~$ nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
```

Send a test guess:

```
<bandit24_password> 0000
Wrong! Please enter the correct pincode. Try again.
```

Two things to note: the daemon stays connected and prompts you to try again, and wrong guesses produce the word `Wrong`. The successful response will be different — it will contain the password. That difference is what you filter for.

Exit with `Ctrl+C`.

---

### Step 3 — Create a Working Directory

```bash
bandit24@bandit:~$ mkdir /tmp/mydir25
bandit24@bandit:~$ cd /tmp/mydir25
```

---

### Step 4 — Write the Brute-Force Script

```bash
bandit24@bandit:/tmp/mydir25$ nano bruteforce.sh
```

Enter the following:

```bash
#!/bin/bash

password=$(cat /etc/bandit_pass/bandit24)

for pin in $(seq -w 0 9999); do
    echo "$password $pin"
done
```

Save and exit.

What each part does:

|Part|What it does|
|---|---|
|`password=$(cat /etc/bandit_pass/bandit24)`|Reads the current level's password from the password file once, storing it in a variable.|
|`seq -w 0 9999`|Generates numbers from 0 to 9999. The `-w` flag pads with leading zeros so all numbers are 4 digits wide: `0000`, `0001`, …, `9999`.|
|`echo "$password $pin"`|Prints one candidate line per iteration in the format the daemon expects.|

The script produces 10,000 lines — one per possible pin — to stdout.

---

### Step 5 — Generate the Candidate List

Make the script executable and run it to generate a file of all candidates:

```bash
bandit24@bandit:/tmp/mydir25$ chmod +x bruteforce.sh
bandit24@bandit:/tmp/mydir25$ ./bruteforce.sh > candidates.txt
bandit24@bandit:/tmp/mydir25$ wc -l candidates.txt
10000 candidates.txt
```

`wc -l` confirms all 10,000 lines were written. Generating the file first (rather than piping the script directly into `nc`) is cleaner — if something goes wrong with the connection you can re-run the `nc` command without regenerating the candidates.

---

### Step 6 — Send All Candidates and Filter the Response

Pipe the candidate file into `nc` and filter out the rejection lines:

```bash
bandit24@bandit:/tmp/mydir25$ cat candidates.txt | nc localhost 30002 | grep -v Wrong
```

What each part does:

|Part|What it does|
|---|---|
|`cat candidates.txt`|Streams all 10,000 candidate lines to stdout.|
|`nc localhost 30002`|Sends them to the daemon over a single persistent connection and returns all responses.|
|`grep -v Wrong`|Suppresses every line containing `Wrong`, leaving only the opening banner and the successful response.|

This will take up to a minute to complete. When the daemon finds the correct pin, it responds with the `bandit25` password. That line passes through `grep -v Wrong` and appears in your terminal.

```
Correct! The password for bandit25 is <password>
```

That is the password for [Bandit Level 25 → Level 26](bandit-level-25-level-26.md).

---

### Step 7 — Save the Password and Exit

> **Warning:** OverTheWire does not remember your progress. Copy the password now with **`Ctrl + Shift + C`** before closing the session.

> **Note:** Per [OverTheWire's rules](https://overthewire.org/rules/), passwords are not published in walkthroughs.

```bash
bandit24@bandit:/tmp/mydir25$ exit
```

---

## Key Takeaways

- **Brute-forcing** means exhaustively trying every possible input. It is the last resort when there is no smarter attack — and sometimes, as here, it is the intended solution.
- **`seq -w 0 9999`** generates a zero-padded numeric sequence. The `-w` flag makes all numbers the same width as the largest, ensuring `0001` not `1`. This matters when the target expects exactly 4 digits.
- **A single persistent connection** is far more efficient than opening a new TCP connection per guess. `nc` holds the connection open as long as stdin is open — piping a file into it sends all lines over one connection.
- **`grep -v pattern`** inverts the match, suppressing lines that contain the pattern and passing everything else. When one response among thousands is different, filtering out the known-bad response is simpler than describing the unknown-good one.
- **Generate data to a file before sending it** — separating generation from transmission makes each step independently repeatable. If the `nc` command fails, you do not need to regenerate 10,000 lines.

---
