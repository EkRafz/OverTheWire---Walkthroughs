

```
			██████╗  █████╗ ███╗   ██╗██████╗ ██╗████████╗
			██╔══██╗██╔══██╗████╗  ██║██╔══██╗██║╚══██╔══╝
			██████╔╝███████║██╔██╗ ██║██║  ██║██║   ██║
			██╔══██╗██╔══██║██║╚██╗██║██║  ██║██║   ██║
			██████╔╝██║  ██║██║ ╚████║██████╔╝██║   ██║
			╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═════╝ ╚═╝   ╚═╝
```


---

## What is Bandit?

Bandit is the recommended starting point for anyone new to Linux or security. Each level presents a simple goal, **find the password hidden somewhere on the server**, and the challenge is figuring out which tools and techniques get you there.

By the end of Bandit you will be comfortable **navigating** the filesystem, **reading** and **searching** files, **understanding** permissions, **working** with pipes and redirects, and **solving** problems using `man` pages.

---

## Quick Start

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

| Field    | Value                         |
| -------- | ----------------------------- |
| Host     | `bandit.labs.overthewire.org` |
| Port     | `2220`                        |
| Username | `bandit0`                     |
| Password | `bandit0`                     |

Never used SSH before? Start with the [connection guide](Walkthroughs/how-to-connect-to-the-server.md).

---

## Levels

| Level                                                                    | Skills Introduced                                          | Status |
| ------------------------------------------------------------------------ | ---------------------------------------------------------- | ------ |
| [Level 0 - Connect to ssh](Walkthroughs/how-to-connect-to-the-server.md) | `ssh`, `-p` flag, host fingerprints, `man`                 | ✅      |
| [Level 0 → 1](Walkthroughs/bandit-level-0-level-1.md)                    | `ls`, `cat`                                                | ✅      |
| [Level 1 → 2](Walkthroughs/bandit-level-1-level-2.md)                    | Files named `-`, `./` prefix, `Ctrl+C`                     | ✅      |
| [Level 2 → 3](Walkthroughs/bandit-level-2-level-3.md)                    | Filenames with spaces, escaping with `\`, quoting          | ✅      |
| [Level 3 → 4](Walkthroughs/bandit-level-3-level-4.md)                    | Hidden files, `ls -la`, file metadata, `cd`                | ✅      |
| [Level 4 → 5](Walkthroughs/bandit-level-4-level-5.md)                    | `file` command, glob wildcards `*`, binary vs ASCII        | ✅      |
| [Level 5 → 6](Walkthroughs/bandit-level-5-level-6.md)                    | `find` by size/type/permissions, pipes `\|`, `grep`        | ✅      |
| [Level 6 → 7](Walkthroughs/bandit-level-6-level-7.md)                    | `find` across `/`, stderr, `2>/dev/null`                   | ✅      |
| [Level 7 → 8](Walkthroughs/bandit-level-7-level-8.md)                    | `grep`, `wc -l`, searching large files                     | ✅      |
| [Level 8 → 9](Walkthroughs/bandit-level-8-level-9.md)                    | `sort`, `uniq`, finding the unique string                  | ✅      |
| [Level 9 → 10](Walkthroughs/bandit-level-9-level-10.md)                  | `strings`, non human-readable files                        | ✅      |
| [Level 10 → 11](Walkthroughs/bandit-level-10-level-11.md)                | `base64`, `-d` to decode data                              | ✅      |
| [Level 11 → 12](Walkthroughs/bandit-level-11-level-12.md)                | `tr`, ROT13 decoding                                       | ✅      |
| [Level 12 → 13](Walkthroughs/bandit-level-12-level-13.md)                | `xxd`, `tar`, `gzip`, `bzip2` decompression                | ✅      |
| [Level 13 → 14](Walkthroughs/bandit-level-13-level-14.md)                | `chmod`, `scp`, SSH auth keys                              | ✅      |
| [Level 14 → 15](Walkthroughs/bandit-level-14-level-15.md)                | `nc`, localhost, ports                                     | ✅      |
| [Level 15 → 16](Walkthroughs/bandit-level-15-level-16.md)                | `openssl s_client -connect host:port`                      | ✅      |
| [Level 16 → 17](Walkthroughs/bandit-level-16-level-17.md)                | `nmap` port scanning, `openssl s_client`                   | ✅      |
| [Level 17 → 18](Walkthroughs/bandit-level-17-level-18.md)                | `diff` — compare two files line by line                    | ✅      |
| [Level 18 → 19](Walkthroughs/bandit-level-18-level-19.md)                | `ssh` — run a single command on a remote machine           | ✅      |
| [Level 19 → 20](Walkthroughs/bandit-level-19-level-20.md)                | setuid binaries, temporary privilege escalation            | ✅      |
| [Level 20 → 21](Walkthroughs/bandit-level-20-level-21.md)                | `nc` listener in background, job control with `&`          | ✅      |
| [Level 21 → 22](Walkthroughs/bandit-level-21-level-22.md)                | `cron`, reading crontabs, following script output          | ✅      |
| [Level 22 → 23](Walkthroughs/bandit-level-22-level-23.md)                | `md5sum`, `cut`, reverse-engineering computed filenames    | ✅      |
| [Level 23 → 24](Walkthroughs/bandit-level-23-level-24.md)                | Writing shell scripts, `chmod +x`, cron drop zones         | ✅      |
| [Level 24 → 25](Walkthroughs/bandit-level-24-level-25.md)                | Brute-forcing, `seq -w`, single persistent `nc` connection | ✅      |
| [Level 25 → 26](Walkthroughs/bandit-level-25-level-26.md)                | `/etc/passwd` shells, `more` pager escape, `vi` `:shell`   | ✅      |
| [Level 26 → 27](Walkthroughs/bandit-level-26-level-27.md)                | setuid binary (revisited), chaining levels                 | ✅      |
| [Level 27 → 28](Walkthroughs/bandit-level-27-level-28.md)                | `git clone` over SSH, exploring a repository               | ✅      |
| [Level 28 → 29](Walkthroughs/bandit-level-28-level-29.md)                | `git log`, `git show`, secrets in commit history           | ✅      |
| [Level 29 → 30](Walkthroughs/bandit-level-29-level-30.md)                | `git branch -a`, `git checkout`, secrets in branches       | ✅      |
| [Level 30 → 31](Walkthroughs/bandit-level-30-level-31.md)                | `git tag`, `git show`, secrets in annotated tags           | ✅      |
| [Level 31 → 32](Walkthroughs/bandit-level-31-level-32.md)                | `git add -f`, `.gitignore`, `git push`, server-side hooks  | ✅      |
| [Level 32 → 33](Walkthroughs/bandit-level-32-level-33.md)                | Uppercase shell escape, `$0`, shell special variables      | ✅      |
| [Level 33 → 34](./Walkthroughs/bandit-level-33-level-34)                 | Final                                                      | ✅      |

---

## Structure

```
bandit/
│
├── bandit-README.md               ← You are here
└── Walkthroughs/
    ├── how-to-connect-to-the-server.md
    ├── bandit-level-0-level-1.md
    ├── bandit-level-1-level-2.md
    └── ...
```

---

