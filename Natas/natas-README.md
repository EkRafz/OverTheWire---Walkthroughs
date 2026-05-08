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

| Level                                                  | Skills Introduced                                           | Status |
| ------------------------------------------------------ | ----------------------------------------------------------- | ------ |
| [Level 0](Walkthroughs/natas-level-0.md)               | View Page Source, `Ctrl + U`                                | ✅      |
| [Level 0 → 1](Walkthroughs/natas-level-0-level-1.md)   | Same a [Level 0](Walkthroughs/natas-level-0.md)             | ✅      |
| [Level 1 → 2](Walkthroughs/natas-level-1-level-2.md)   | Directory listing to navigate directories                   | ✅      |
| [Level 2 → 3](Walkthroughs/natas-level-2-level-3.md)   | `robots.txt`, publicly accessible file that lists paths     | ✅      |
| [Level 3 → 4](Walkthroughs/natas-level-3-level-4.md)   | `curl`, send custom requests, http headers, `referrer`      | ✅      |
| [Level 4 → 5](Walkthroughs/natas-level-4-level-5.md)   | `curl`, edit login cookies                                  | ✅      |
| [Level 5 → 6](Walkthroughs/natas-level-5-level-6.md)   | `php` script, file navigation `.inc`                        | ✅      |
| [Level 6 → 7](Walkthroughs/natas-level-6-level-7.md)   | Bypass directory with custom url                            | ✅      |
| [Level 7 → 8](Walkthroughs/natas-level-7-level-8.md)   | encoding/decoding, `php`, cyberchef                         | ✅      |
| [Level 8 → 9](Walkthroughs/natas-level-8-level-9.md)   | Command injection, `;`, no sanitation, `grep`               | ✅      |
| [Level 9 → 10](Walkthroughs/natas-level-9-level-10.md) | `grep`, `#`, root vulnerability, non- exhaustive sanitation | ✅      |
| Level 10 → 11                                          |                                                             | 🔜     |
| Level 11 → 12                                          |                                                             | 🔜     |
| Level 12 → 13                                          |                                                             | 🔜     |
| Level 13 → 14                                          |                                                             | 🔜     |
| Level 14 → 15                                          |                                                             | 🔜     |
| Level 15 → 16                                          |                                                             | 🔜     |
| Level 16 → 17                                          |                                                             | 🔜     |
| Level 17 → 18                                          |                                                             | 🔜     |
| Level 18 → 19                                          |                                                             | 🔜     |
| Level 19 → 20                                          |                                                             | 🔜     |
| Level 20 → 21                                          |                                                             | 🔜     |
| Level 21 → 22                                          |                                                             | 🔜     |
| Level 22 → 23                                          |                                                             | 🔜     |
| Level 23 → 24                                          |                                                             | 🔜     |
| Level 24 → 25                                          |                                                             | 🔜     |
| Level 25 → 26                                          |                                                             | 🔜     |
| Level 26 → 27                                          |                                                             | 🔜     |
| Level 27 → 28                                          |                                                             | 🔜     |
| Level 28 → 29                                          |                                                             | 🔜     |
| Level 29 → 30                                          |                                                             | 🔜     |
| Level 30 → 31                                          |                                                             | 🔜     |
| Level 31 → 32                                          |                                                             | 🔜     |
| Level 32 → 33                                          |                                                             | 🔜     |
| Level 33 → 34                                          |                                                             | 🔜     |

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

| Bandit                        | Natas                            |
| ----------------------------- | -------------------------------- |
| SSH access to a Linux server  | HTTP access to a web application |
| Filesystem / shell challenges | Web vulnerability challenges     |
| Tools: `ls`, `cat`, `grep`…   | Tools: browser DevTools, `curl`  |
| Teaches Linux fundamentals    | Teaches server-side web security |

---
