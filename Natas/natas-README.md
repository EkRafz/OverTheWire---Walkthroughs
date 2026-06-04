```
			███╗   ██╗ █████╗ ████████╗ █████╗ ███████╗
			████╗  ██║██╔══██╗╚══██╔══╝██╔══██╗██╔════╝
			██╔██╗ ██║███████║   ██║   ███████║███████╗
			██║╚██╗██║██╔══██║   ██║   ██╔══██║╚════██║
			██║ ╚████║██║  ██║   ██║   ██║  ██║███████║
			╚═╝  ╚═══╝╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚══════╝
```

---

## What is Natas?

Natas teaches the basics of **server-side web security**. Unlike Bandit, there is no SSH — each level is its own website. Your job is always the same: **find the password for the next level** hidden somewhere on the current page.

Passwords are stored in `/etc/natas_webpass/natasX` on the server. You won't get there directly; instead you exploit whatever vulnerability the level exposes to read it.

By the end of Natas you will be comfortable **reading page source**, **manipulating HTTP headers and cookies**, **exploiting command and SQL injection**, **working with encoding and crypto**, and tackling advanced topics like **PHP object injection**, **Perl CGI bugs**, and **PHAR deserialization**.

> I **did not** use any special tools like `burp suite` or anything similar to that. This walkthrough purely uses tools that are available in most linux distros like `php`, `python`, `cURL` and browser tools.

---

## Quick Start

|Field|Value|
|---|---|
|URL|`http://natas0.natas.labs.overthewire.org`|
|Username|`natas0`|
|Password|`natas0`|

Each subsequent level follows the pattern `http://natasX.natas.labs.overthewire.org` where `X` is the level number. Log in with the username and password you found on the previous level.

---

## Levels

| Level                                                    | Skills Introduced                                              | Status |
| -------------------------------------------------------- | -------------------------------------------------------------- | ------ |
| [Level 0](Walkthroughs/natas-level-0.md)                 | View Page Source, `Ctrl + U`                                   | ✅      |
| [Level 0 → 1](Walkthroughs/natas-level-0-level-1.md)     | Same a [Level 0](Walkthroughs/natas-level-0.md)                | ✅      |
| [Level 1 → 2](Walkthroughs/natas-level-1-level-2.md)     | Directory listing to navigate directories                      | ✅      |
| [Level 2 → 3](Walkthroughs/natas-level-2-level-3.md)     | `robots.txt`, publicly accessible file that lists paths        | ✅      |
| [Level 3 → 4](Walkthroughs/natas-level-3-level-4.md)     | `curl`, send custom requests, http headers, `referrer`         | ✅      |
| [Level 4 → 5](Walkthroughs/natas-level-4-level-5.md)     | `curl`, edit login cookies                                     | ✅      |
| [Level 5 → 6](Walkthroughs/natas-level-5-level-6.md)     | `php` script, file navigation `.inc`                           | ✅      |
| [Level 6 → 7](Walkthroughs/natas-level-6-level-7.md)     | Bypass directory with custom url                               | ✅      |
| [Level 7 → 8](Walkthroughs/natas-level-7-level-8.md)     | encoding/decoding, `php`, cyberchef                            | ✅      |
| [Level 8 → 9](Walkthroughs/natas-level-8-level-9.md)     | Command injection, `;`, no sanitization, `grep`                | ✅      |
| [Level 9 → 10](Walkthroughs/natas-level-9-level-10.md)   | `grep`, `#`, root vulnerability, non- exhaustive sanitization  | ✅      |
| [Level 10 → 11](Walkthroughs/natas-level-10-level-11.md) | XOR encryption, cookie manipulation, base64                    | ✅      |
| [Level 11 → 12](Walkthroughs/natas-level-11-level-12.md) | File upload, extension bypass, content-type                    | ✅      |
| [Level 12 → 13](Walkthroughs/natas-level-12-level-13.md) | File upload, `exif_imagetype()` bypass, webshell               | ✅      |
| [Level 13 → 14](Walkthroughs/natas-level-13-level-14.md) | SQL injection, `"` escape, `OR 1=1`                            | ✅      |
| [Level 14 → 15](Walkthroughs/natas-level-14-level-15.md) | Blind SQLi, `SUBSTRING()`, `ORD()`, Python automation          | ✅      |
| [Level 15 → 16](Walkthroughs/natas-level-15-level-16.md) | Command injection, `grep`, subshell `$()`, charset brute-force | ✅      |
| [Level 16 → 17](Walkthroughs/natas-level-16-level-17.md) | Time-based blind SQLi, `SLEEP()`, `IF()`, binary search        | ✅      |
| [Level 17 → 18](Walkthroughs/natas-level-17-level-18.md) | Session brute-force, sequential session IDs, `PHPSESSID`       | ✅      |
| [Level 18 → 19](Walkthroughs/natas-level-18-level-19.md) | Session brute-force, encoded session IDs, `bin2hex()`          | ✅      |
| [Level 19 → 20](Walkthroughs/natas-level-19-level-20.md) | Custom session handling, newline injection, `$_SESSION` write  | ✅      |
| [Level 20 → 21](Walkthroughs/natas-level-20-level-21.md) | Colocated sites, cross-site session injection, `foreach` loop  | ✅      |
| [Level 21 → 22](Walkthroughs/natas-level-21-level-22.md) | `header()` without `exit()`, redirect body bypass, `curl`      | ✅      |
| [Level 22 → 23](Walkthroughs/natas-level-22-level-23.md) | PHP type juggling, `strstr()`, string-to-int coercion          | ✅      |
| [Level 23 → 24](Walkthroughs/natas-level-23-level-24.md) | `strcmp()` array bypass, type mismatch, `NULL` return          | ✅      |
| [Level 24 → 25](Walkthroughs/natas-level-24-level-25.md) |                                                                | ✅      |
| [Level 25 → 26](Walkthroughs/natas-level-25-level-26.md) |                                                                | ✅      |
| [Level 26 → 27](Walkthroughs/natas-level-26-level-27.md) |                                                                | ✅      |
| [Level 27 → 28](Walkthroughs/natas-level-27-level-28.md) |                                                                | ✅      |
| [Level 28 → 29](Walkthroughs/natas-level-28-level-29.md) |                                                                | ✅      |
| [Level 29 → 30](Walkthroughs/natas-level-29-level-30.md) |                                                                | ✅      |
| [Level 30 → 31](Walkthroughs/natas-level-30-level-31.md) |                                                                | ✅      |
| [Level 31 → 32](Walkthroughs/natas-level-31-level-32.md) |                                                                | ✅      |
| [Level 32 → 33](Walkthroughs/natas-level-32-level-33.md) |                                                                | ✅      |
| [Level 33 → 34](Walkthroughs/natas-level-33-level-34.md) |                                                                | ✅      |

---

## Structure

```
Natas/
│
├── natas-README.md               ← You are here
└── Walkthroughs/
    ├── natas-level-0.md
    ├── natas-level-0-level-1.md
    ├── natas-level-1-level-2.md
    └── ...
```

---

## Key Differences from Bandit

| Bandit                        | Natas                                            |
| ----------------------------- | ------------------------------------------------ |
| SSH access to a Linux server  | HTTP access to a web application                 |
| Filesystem / shell challenges | Web vulnerability challenges                     |
| Tools: `ls`, `cat`, `grep`…   | Tools: browser DevTools, `curl`, `php`, `python` |
| Teaches Linux fundamentals    | Teaches server-side web security                 |

---
