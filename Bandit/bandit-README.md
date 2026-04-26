

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

| Level                                                                    | Skills Introduced                                   | Status |
| ------------------------------------------------------------------------ | --------------------------------------------------- | ------ |
| [Level 0 - Connect to ssh](Walkthroughs/how-to-connect-to-the-server.md) | ssh, `-p` flag, host fingerprints, `man`            | ✅      |
| [Level 0 → 1](Walkthroughs/bandit-level-0-level-1.md)                    | `ls`, `cat`                                         | ✅      |
| [Level 1 → 2](Walkthroughs/bandit-level-1-level-2.md)                    | Files named `-`, `./` prefix, `Ctrl+C`              | ✅      |
| [Level 2 → 3](Walkthroughs/bandit-level-2-level-3.md)                    | Filenames with spaces, escaping with `\`, quoting   | ✅      |
| [Level 3 → 4](Walkthroughs/bandit-level-3-level-4.md)                    | Hidden files, `ls -la`, file metadata, `cd`         | ✅      |
| [Level 4 → 5](Walkthroughs/bandit-level-4-level-5.md)                    | `file` command, glob wildcards `*`, binary vs ASCII | ✅      |
| [Level 5 → 6](Walkthroughs/bandit-level-5-level-6.md)                    | `find` by size/type/permissions, pipes `\|`, `grep` | ✅      |
| [Level 6 → 7](Walkthroughs/bandit-level-6-level-7.md)                    | `find` across `/`, stderr, `2>/dev/null`            | ✅      |
| [Level 7 → 8](Walkthroughs/bandit-level-7-level-8.md)                    | `grep`, `wc -l`, searching large files              | ✅      |
| [level 8→ 9](Walkthroughs/bandit-level-8-level-9.md)                     | `sort`, `uniq`, searching unique string             | ✅      |
| [level 8→ 9](Walkthroughs/bandit-level-9-level-10.md)                    | `strings`, non human-readable files                 | ✅      |
| Level 10 → 11                                                            | —                                                   | 🔜     |
| Level 11 → 12                                                            | —                                                   | 🔜     |
| Level 12 → 13                                                            | —                                                   | 🔜     |
| Level 13 → 14                                                            | —                                                   | 🔜     |
| Level 14 → 15                                                            | —                                                   | 🔜     |
| Level 15 → 16                                                            | —                                                   | 🔜     |
| Level 16 → 17                                                            | —                                                   | 🔜     |
| Level 17 → 18                                                            | —                                                   | 🔜     |
| Level 18 → 19                                                            | —                                                   | 🔜     |
| Level 19 → 20                                                            | —                                                   | 🔜     |
| Level 20 → 21                                                            | —                                                   | 🔜     |
| Level 21 → 22                                                            | —                                                   | 🔜     |
| Level 22 → 23                                                            | —                                                   | 🔜     |
| Level 23 → 24                                                            | —                                                   | 🔜     |
| Level 24 → 25                                                            | —                                                   | 🔜     |
| Level 25 → 26                                                            | —                                                   | 🔜     |
| Level 26 → 27                                                            | —                                                   | 🔜     |
| Level 27 → 28                                                            | —                                                   | 🔜     |
| Level 28 → 29                                                            | —                                                   | 🔜     |
| Level 29 → 30                                                            | —                                                   | 🔜     |
| Level 30 → 31                                                            | —                                                   | 🔜     |
| Level 31 → 32                                                            | —                                                   | 🔜     |
| Level 32 → 33                                                            | —                                                   | 🔜     |
| Level 33 → 34                                                            | —                                                   | 🔜     |

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

