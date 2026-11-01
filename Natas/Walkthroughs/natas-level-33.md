> **Level Goal:** At this moment, level 34 does not exist yet.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas34.natas.labs.overthewire.org`                  |
| Username | `natas34`                                                    |
| Password | *found in [Natas Level 32 → 33](natas-level-32-level-33.md)* |

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas34.natas.labs.overthewire.org` and log in with the password from Level 33.

---

### Step 2 — Read the Page

```
Congratulations! You have reached the end... for now.
```

That's it. You've completed Natas.

---

## What You've Covered

Across 34 levels, Natas introduced the core concepts of web application security:

|Category|Topics|
|---|---|
|**Recon**|Page source, HTML comments, robots.txt, directory listing, HTTP headers|
|**Auth bypass**|Cookie manipulation, session tampering, referrer spoofing|
|**Encoding**|Base64, XOR, hex, custom encoding schemes|
|**Injection**|Command injection, SQL injection, blind SQLi, timing attacks, second-order injection|
|**File handling**|Upload bypass, MIME type spoofing, null-byte injection, path traversal|
|**PHP quirks**|Type juggling, loose comparison, `unserialize()`, object injection, PCRE limits, MySQL truncation|
|**Crypto**|AES-ECB block splicing|
|**Perl**|Two-argument `open()`, `DBI::quote()` type confusion, `CGI::param()` list flattening, `ARGV` filehandle hijack|
|**Deserialization**|Phar metadata deserialization via file operations, `__destruct()` gadgets|

---