
[![CC BY 4.0][cc-by-shield]][cc-by]
 
[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg

---

```
						 ██████╗ ████████╗██╗    ██╗
						██╔═══██╗╚══██╔══╝██║    ██║
						██║   ██║   ██║   ██║ █╗ ██║
						██║   ██║   ██║   ██║███╗██║
						╚██████╔╝   ██║   ╚███╔███╔╝
						 ╚═════╝    ╚═╝    ╚══╝╚══╝ 
```

---

## What is OverTheWire?

[OverTheWire](https://overthewire.org/wargames/) is a platform that teaches Linux and security concepts through **wargames**, **terminal-based challenges** where you find passwords to unlock the next level. No GUI, no hints, just you and a shell.

Each wargame targets a different skill level and topic, building from basic command-line navigation all the way up to binary exploitation and reverse engineering.

---

## How to Use This Repo

Each wargame has its own folder containing an index and one file per level. Every walkthrough follows the same structure: connection details, commands introduced, a step-by-step solution with explanations, and key takeaways.

**Try the level yourself before opening the walkthrough.** The struggle is where the learning happens.

> **Passwords are never published here**, per [OverTheWire's rules](https://overthewire.org/rules/). The walkthroughs explain *how* to find the password, not what it is.

---

## Wargames Index

| Wargame                             | Difficulty              | Topic                                    | Status                       |
| ----------------------------------- | ----------------------- | ---------------------------------------- | ---------------------------- |
| [Bandit](./Bandit/bandit-README.md) | Beginner                | Linux fundamentals, SSH, shell basics    | 🟢 Completed (Levels 0–34)   |
| [Natas](./Natas/natas-README.md)    | Beginner                | Web security, PHP, SQL injection         | 🟠 In Progress (levels 0-29) |
| Leviathan                           | Beginner                | Basic exploitation, SUID binaries        | 🔴 Coming Soon               |
| Krypton                             | Beginner → Intermediate | Cryptography and ciphers                 | 🔴 Coming Soon               |
| Narnia                              | Intermediate            | C binary exploitation, memory corruption | 🔴 Coming Soon               |
| Behemoth                            | Intermediate            | Binary exploitation, shellcode           | 🔴 Coming Soon               |
| Utumno                              | Intermediate            | Advanced binary exploitation             | 🔴 Coming Soon               |
| Maze                                | Intermediate            | Reverse engineering                      | 🔴 Coming Soon               |
| Vortex                              | Advanced                | Shellcoding, format strings              | 🔴 Coming Soon               |
| Manpage                             | Advanced                | Linux internals                          | 🔴 Coming Soon               |
| Drifter                             | Advanced                | Heap exploitation                        | 🔴 Coming Soon               |
| Formula                             | Advanced                | Advanced exploitation                    | 🔴 Coming Soon               |

---

## Repository Structure

```
overthewire/
│
├── README.md               ← You are here
│
└── Bandit/
    ├── bandit-README.md           ← Bandit index and level table
    └── Walkthroughs/
        ├── how-to-connect-to-the-server.md
        ├── bandit-level-0-level-1.md
        └── ...
```

New wargames will each get their own top-level folder following the same layout.

---

## Prerequisites

A terminal with SSH. That's it for most wargames.

| OS | Setup |
| -- | ----- |
| Linux / macOS | SSH is built in — open a terminal and go |
| Windows | [Windows Terminal](https://aka.ms/terminal) + [OpenSSH](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse), or use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) |

---

## Useful Resources

| Resource | What it's for |
| -------- | ------------- |
| [explainshell.com](https://explainshell.com/) | Break down any shell command flag by flag |
| [tldr.sh](https://tldr.sh/) | Practical, example-driven man pages |
| [The Linux Command Line (free)](https://linuxcommand.org/tlcl.php) | Best beginner book for the terminal |
| [OverTheWire official site](https://overthewire.org/wargames/) | Game descriptions and hints |

---

## Licence

Shield: [![CC BY 4.0][cc-by-shield]][cc-by]

This work is licensed under a
[Creative Commons Attribution 4.0 International License][cc-by].

© 2026 [EkRafz](https://github.com/EkRafz). All walkthroughs, explanations, and content in this repository are original work by EkRafz.

[![CC BY 4.0][cc-by-image]][cc-by]

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg
