```
	██╗     ███████╗██╗   ██╗██╗ █████╗ ████████╗██╗  ██╗ █████╗ ███╗   ██╗
	██║     ██╔════╝██║   ██║██║██╔══██╗╚══██╔══╝██║  ██║██╔══██╗████╗  ██║
	██║     █████╗  ██║   ██║██║███████║   ██║   ███████║███████║██╔██╗ ██║
	██║     ██╔══╝  ╚██╗ ██╔╝██║██╔══██║   ██║   ██╔══██║██╔══██║██║╚██╗██║
	███████╗███████╗ ╚████╔╝ ██║██║  ██║   ██║   ██║  ██║██║  ██║██║ ╚████║
	╚══════╝╚══════╝  ╚═══╝  ╚═╝╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝
```

---

## What is Leviathan?

Leviathan bridges the gap between **Bandit** (Linux basics) and more advanced wargames. Each level drops you into a home directory with a suspicious SUID binary, your job is always the same: **get the password for the next level** stored in `/etc/leviathan_pass/leviathanX`.

No programming knowledge is required. The core loop is: inspect what's there, figure out what the binary does, find the flaw, exploit it.

By the end of Leviathan you will be comfortable **tracing binary execution** with `ltrace` and `strace`, **understanding SUID privilege escalation**, **exploiting symlinks**, **brute-forcing 4-digit codes**, **reading binary output**, and thinking adversarially about how small mistakes in programs become vulnerabilities.

> Like the Bandit and Natas walkthroughs, this one uses only standard Linux tools: `ltrace`, `strace`, `strings`, `file`, `ln`, `grep`, and `bash`, no external frameworks needed.

---

## Quick Start

```bash
ssh leviathan0@leviathan.labs.overthewire.org -p 2223
```

|Field|Value|
|---|---|
|Host|`leviathan.labs.overthewire.org`|
|Port|`2223`|
|Username|`leviathan0`|
|Password|`leviathan0`|

Each subsequent level follows the pattern `leviathanX` as the username. Log in with the password you found on the previous level.

---

## Levels

| Level                                                      | Skills Introduced                                         | Status |
| ---------------------------------------------------------- | --------------------------------------------------------- | ------ |
| [Level 0](leviathan-level-0.md)                            | `ssh`, `-p` flag, initial recon                           | ✅      |
| [Level 0 → 1](./Walkthroughs/leviathan-level-0-level-1.md) | Hidden directories, `ls -la`, `grep` through backup files | ✅      |
| [Level 1 → 2](./Walkthroughs/leviathan-level-1-level-2.md) |                                                           | 🔜     |
| [Level 2 → 3](./Walkthroughs/leviathan-level-2-level-3.md) |                                                           | 🔜     |
| [Level 3 → 4](./Walkthroughs/leviathan-level-3-level-4.md) |                                                           | 🔜     |
| [Level 4 → 5](./Walkthroughs/leviathan-level-4-level-5.md) |                                                           | 🔜     |
| [Level 5 → 6](./Walkthroughs/leviathan-level-5-level-6.md) |                                                           | 🔜     |
| [Level 6 → 7](./Walkthroughs/leviathan-level-6-level-7.md) |                                                           | 🔜     |
| [Level 7](./Walkthroughs/leviathan-level-7.md)             |                                                           | 🔜     |

---

## Structure

```
Leviathan/
│
├── leviathan-README.md               ← You are here
└── Walkthroughs/
    ├── leviathan-level-0.md
    ├── leviathan-level-0-level-1.md
    ├── leviathan-level-1-level-2.md
    └── ...
```

---

## Key Differences from Bandit and Natas

|Bandit|Natas|Leviathan|
|---|---|---|
|SSH, Linux filesystem basics|HTTP, server-side web security|SSH, binary analysis and privilege escalation|
|Tools: `ls`, `cat`, `grep`…|Tools: browser, `curl`, `php`…|Tools: `ltrace`, `strace`, `strings`, `ln`…|
|Teaches shell fundamentals|Teaches web vulnerabilities|Teaches binary exploitation fundamentals|
|No programming needed|No programming needed|No programming needed|

---

## Core Tools You Will Use

|Tool|What It Does|
|---|---|
|`ltrace`|Traces library calls at runtime — the main tool for this game|
|`strace`|Traces system calls — useful for seeing file opens and reads|
|`strings`|Dumps printable strings from a binary — first pass recon|
|`file`|Identifies file type — always run this on unknowns|
|`ln -s`|Creates symlinks — key to the file-redirection exploits|
|`find`|Locates SUID binaries and hidden files across the filesystem|

---