> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| URL      | `http://natas7.natas.labs.overthewire.org`               |
| Username | `natas7`                                                 |
| Password | *found in [Natas Level 5 → 6](natas-level-5-level-6.md)* |

---

## Tools Used

- A web browser
- **View Page Source** — read HTML comments left by the developer
- **URL manipulation** — crafting a query string to exploit the vulnerable parameter

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas7.natas.labs.overthewire.org` and log in with the password from Level 6.

The page displays two links: **Home** and **About**. Clicking either one loads content and updates the URL:

```
http://natas7.natas.labs.overthewire.org/index.php?page=home
http://natas7.natas.labs.overthewire.org/index.php?page=about
```

The `?page=` parameter is controlling which file the server loads and renders. That is the attack surface.

---

### Step 2 — Read the Page Source

View the source with `Ctrl + U`. Both pages contain this HTML comment:

```html
<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
```

The password file is at a known absolute path on the server's filesystem. The question is whether the `?page=` parameter can be pointed at it.

---

### Understand Local File Inclusion (LFI)

The server is using PHP's `include` (or equivalent) to load the file named by the `page` parameter and render its contents into the page. A simplified version of what the PHP likely looks like:

```php
<?php
include($_GET['page']);
?>
```

When `page=home`, the server includes the file `home` (or `home.php`). When `page=about`, it includes `about`.

**Local File Inclusion (LFI)** is a vulnerability that occurs when user-supplied input is passed directly to a file-loading function without validation or sanitisation. The server will include whatever path you give it — not just web pages, but any file on the filesystem the web server process has permission to read.

The `?page=` parameter here has no input validation. Nothing stops you from supplying an absolute path like `/etc/natas_webpass/natas8` instead of a page name.

This is distinct from the previous level's `.inc` file fetch. There, the file was passively accessible at a known URL. Here, you are actively abusing the server's own file-loading logic to read a file that has no URL at all — it exists only on the server's local filesystem.

---

### Understand /etc/natas_webpass/

All Natas passwords are stored as files under `/etc/natas_webpass/`. Each file is named after the user whose password it contains:

```
/etc/natas_webpass/natas8   ← contains the natas8 password
/etc/natas_webpass/natas9   ← contains the natas9 password
```

File permissions restrict direct access — `natas8`'s password file is readable by the `natas7` and `natas8` system users, but not by others. The web server process for natas7 runs as the `natas7` user, so it has permission to read `/etc/natas_webpass/natas8`. The LFI vulnerability lets you leverage those permissions through the web server itself.

---

### Step 3 — Exploit the LFI

Replace the value of `page` with the absolute path of the password file:

```
http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8
```

The server includes the password file and renders its contents into the page:

```
<password>
```

That is the password for [Natas Level 7 → 8](natas-level-7-level-8.md).

---

## Key Takeaways

- **User input passed to a file-loading function without validation is an LFI vulnerability.** If `?page=home` works, `?page=/etc/passwd` is worth trying. The server cannot tell the difference — both are just strings passed to `include`.
- **Absolute paths bypass directory assumptions.** A developer might assume `page=home` means "load `home.php` from the web root." Supplying an absolute path like `/etc/natas_webpass/natas8` steps outside that assumption entirely.
- **Developer comments in HTML source are reconnaissance.** The hint here was not hidden — it was left in a comment visible to anyone who views the source. In a real application, a comment pointing to a sensitive file path is a critical information leak.
- **LFI impact depends on filesystem permissions.** The vulnerability only exposes files the web server process can read. On a misconfigured server, that can include config files, application source, SSH keys, and password stores — not just intentionally public files.

---