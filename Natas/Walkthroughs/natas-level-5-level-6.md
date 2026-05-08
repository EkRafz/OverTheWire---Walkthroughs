> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| URL      | `http://natas6.natas.labs.overthewire.org`               |
| Username | `natas6`                                                 |
| Password | *found in [Natas Level 4 → 5](natas-level-4-level-5.md)* |

---

## Tools Used

- A web browser
- **View Page Source** — read the raw HTML and PHP the server exposes
- **URL navigation** — directly browsing to server-side include paths

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas6.natas.labs.overthewire.org` and log in with the password from Level 5.

The page displays an input form:

```
Input secret: [        ] [Submit]
```

There is also a link: **View sourcecode**. Click it.

---

### Understand the Source Code

The page source is a mix of HTML and PHP. Two parts matter:

**The HTML form:**

```html
<form method=post>
    Input secret: <input name=secret><br>
    <input type=submit name=submit>
</form>
```

The form sends a POST request with two variables: `secret` (your input) and `submit` (the button).

**The PHP logic:**

```php
<?
include "includes/secret.inc";

if(array_key_exists("submit", $_POST)) {
    if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>
```

The PHP compares your submitted `secret` against a variable called `$secret`. That variable is never defined in this file, it comes from the `include` statement at the top, which pulls in `includes/secret.inc`.

PHP's `include` drops the contents of another file directly into the current script at that line. Any variables defined in the included file become available in the including script. The variable `$secret` is defined somewhere inside `includes/secret.inc`.

---

### Understand PHP and Server-Side Code

**PHP** is a server-side scripting language embedded in HTML. When a browser requests a `.php` page, the server executes the PHP code and sends only the resulting HTML output — the PHP source never reaches the browser under normal circumstances.

However, this only applies to files the server is configured to execute as PHP. Files with other extensions — like `.inc` — may be served as raw text instead of being executed, depending on server configuration. If the server is not told to treat `.inc` files as PHP, requesting one directly will return the source code rather than running it.

This is the misconfiguration to exploit here.

---

### Step 2 — Request the Include File Directly

The include path in the source is `includes/secret.inc`. Navigate to it directly:

```
http://natas6.natas.labs.overthewire.org/includes/secret.inc
```

The page appears blank, but the server returned the file. View the source with `Ctrl + U`:

```php
<?
$secret = "XXXXXXXXXXXXXXXXXXX";
?>
```

The secret is defined in plain text.

---

### Step 3 — Submit the Secret

Go back to `http://natas6.natas.labs.overthewire.org`, paste the secret into the input field, and submit.

The page returns:

```
Access granted. The password for natas7 is <password>
```

That is the password for [Natas Level 6 → 7](natas-level-6-level-7.md).

---

## Key Takeaways

- **Server-side code is hidden from the browser — but the files it references may not be.** PHP source is never sent to the client directly, but any file the server does not execute gets served as plain text. A misconfigured web server will happily return the raw contents of a `.inc` file, exposing the source inside it.
- **`include` and `require` paths in source code are a map.** Any time you can read the PHP source of a page, look for `include` and `require` statements. They point to other files on the server worth fetching directly.
- **Secrets embedded in source files are not secret.** A hardcoded `$secret = "..."` in an included file is only as protected as the file itself. If the file is reachable over HTTP and the server serves it as text, the secret is public.
- **Extension ≠ execution.** Web servers decide whether to execute a file as PHP based on configuration, not the file's contents. `.php` files are typically executed; `.inc`, `.txt`, and others may not be, even if they contain PHP code.

---