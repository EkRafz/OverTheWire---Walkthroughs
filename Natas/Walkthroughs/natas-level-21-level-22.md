> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas22.natas.labs.overthewire.org`                  |
| Username | `natas22`                                                    |
| Password | *found in [Natas Level 20 → 21](natas-level-20-level-21.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`curl`** — to suppress redirect-following and read the response body

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas22.natas.labs.overthewire.org` and log in with the password from Level 21.

The page is completely blank. No form, no visible content, no instructions. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

The source is two short PHP blocks separated by the HTML body:

```php
<?php
	session_start();

	if(array_key_exists("revelio", $_GET)) {
	    // only admins can reveal the password
	    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
	        header("Location: /");
	    }
}
?>
```

```php
<?php
	if(array_key_exists("revelio", $_GET)) {
	    print "You are an admin. The credentials for the next level are:<br>";
	    print "<pre>Username: natas23\n";
	    print "Password: <censored></pre>";
	}
?>
```

Walking through both blocks together is the key to understanding the vulnerability.

**First block — the guard:**

```php
if(array_key_exists("revelio", $_GET)) {
```

Triggers only when the URL contains `?revelio` (the value doesn't matter; `array_key_exists` checks for the key's presence, not its value).

```php
if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");
}
```

If the current session does not have `admin == 1`, PHP calls `header("Location: /")`. This schedules a redirect header to be sent with the response. The intent is to bounce non-admin visitors away before they can see the password.

**Second block — the output:**

```php
if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas23\n";
    print "Password: <censored></pre>";
}
```

This runs unconditionally after the first block — there is no `else`, no early `exit()`, no `die()`. If `revelio` is in the query string, the password is written into the response body regardless of what happened in the first block.

The two blocks are not connected. The first block may add a `Location` header to the response; the second block adds the password to the body of that same response. The response goes out containing both.

---

### Understand the Vulnerability: `header()` Does Not Stop Execution

`header("Location: /")` does one thing: it adds an HTTP response header that instructs the client to navigate to a different URL. It does **not**:

- Stop PHP execution
- Prevent subsequent code from running
- Clear or suppress the response body

PHP continues executing every line after `header()` is called. In this case, the second block runs, prints the password, and the complete response — status `302 Found`, `Location: /` header, and a body containing the password — is sent to the client.

A browser receiving a `302` immediately follows the redirect. It discards the response body and navigates to `/`, which is why the page appears blank. The browser never displays the body of a redirect response — but the body was transmitted. The password was sent.

This is a structural mistake: treating `header("Location: /")` as equivalent to `exit()`. The correct guard is:

```php
header("Location: /");
exit();
```

Without `exit()`, the redirect is a suggestion to the client, not a halt to the server.

---

### Step 3 — Request the Page with `curl`

A browser automatically follows redirects. `curl` does not — unless passed the `-L` flag. Without `-L`, `curl` prints the raw response body of the first response and stops, regardless of the `Location` header.

```bash
curl -u natas22:<password> \
     "http://natas22.natas.labs.overthewire.org/index.php?revelio"
```

|Flag / argument|Meaning|
|---|---|
|`-u natas22:<password>`|HTTP Basic Authentication — sends credentials in the `Authorization` header|
|`?revelio`|Triggers both PHP blocks — the redirect check and the password output|
|No `-L` flag|`curl` does not follow the `302` redirect; it prints the first response body and exits|

The server responds with `302 Found` plus a body. `curl` prints the body:

```html
<html>
...
You are an admin. The credentials for the next level are:<br>
<pre>Username: natas23
Password: <password></pre>
...
</html>
```

That is the password for [Natas Level 22 → 23](natas-level-22-level-23.md).

---

### Alternative: Burp Suite

If you prefer a browser-based approach, route your browser through Burp Suite's proxy and navigate to:

```
http://natas22.natas.labs.overthewire.org/index.php?revelio
```

In Burp's **HTTP history**, find the `302` response. The **Response** tab shows the full body — including the password — before the browser followed the redirect. Burp captures and displays every response regardless of status code.

This is the same information as `curl` produces, just surfaced through a different tool.

---

## Key Takeaways

- **`header("Location: /")` is not `exit()`.** PHP's `header()` function schedules an HTTP header. It does not terminate script execution, clear the response buffer, or prevent subsequent `print` and `echo` statements from running. Every line after `header("Location: /")` executes normally. The password was written into the response body before the redirect was ever sent.
- **The response body of a redirect is transmitted, just not displayed by browsers.** HTTP does not prohibit a `302` response from having a body. Browsers choose to ignore it because they immediately navigate to the redirect target. Other clients — `curl`, Burp Suite, Python's `requests` with `allow_redirects=False` — read the body exactly as sent.
- **Sensitive output must be gated with `exit()` after a redirect.** The canonical correct pattern is `header("Location: /"); exit();`. The `exit()` call halts PHP execution immediately, ensuring nothing after it can run and nothing additional can be added to the response.
- **Redirects are a client-side convention, not a server-side security boundary.** A server cannot force a client to follow a redirect. Any HTTP client that chooses to ignore the `Location` header will see the full response, including any body the server already wrote.

---