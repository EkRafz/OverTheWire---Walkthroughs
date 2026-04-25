

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

[OverTheWire](https://overthewire.org/wargames/) is a platform that teaches Linux and security concepts through **wargames** — **terminal-based challenges** where you find passwords to unlock the next level. No GUI, no hints, just you and a shell.

Each wargame targets a different skill level and topic, building from basic command-line navigation all the way up to binary exploitation and reverse engineering.

---

## How to Use This Repo

Each wargame has its own folder containing an index and one file per level. Every walkthrough follows the same structure: connection details, commands introduced, a step-by-step solution with explanations, and key takeaways.

**Try the level yourself before opening the walkthrough.** The struggle is where the learning happens.

> **Passwords are never published here**, per [OverTheWire's rules](https://overthewire.org/rules/). The walkthroughs explain *how* to find the password — not what it is.

---

## Wargames Index

| Wargame             | Difficulty              | Topic                                    | Status                      |
| ------------------- | ----------------------- | ---------------------------------------- | --------------------------- |
| [Bandit](./bandit/) | Beginner                | Linux fundamentals, SSH, shell basics    | 🟢 In Progress (Levels 0–8) |
| Leviathan           | Beginner                | Basic exploitation, SUID binaries        | 🔜 Coming Soon              |
| Natas               | Beginner → Intermediate | Web security, PHP, SQL injection         | 🔜 Coming Soon              |
| Krypton             | Beginner → Intermediate | Cryptography and ciphers                 | 🔜 Coming Soon              |
| Narnia              | Intermediate            | C binary exploitation, memory corruption | 🔜 Coming Soon              |
| Behemoth            | Intermediate            | Binary exploitation, shellcode           | 🔜 Coming Soon              |
| Utumno              | Intermediate            | Advanced binary exploitation             | 🔜 Coming Soon              |
| Maze                | Intermediate            | Reverse engineering                      | 🔜 Coming Soon              |
| Vortex              | Advanced                | Shellcoding, format strings              | 🔜 Coming Soon              |
| Manpage             | Advanced                | Linux internals                          | 🔜 Coming Soon              |
| Drifter             | Advanced                | Heap exploitation                        | 🔜 Coming Soon              |
| Formula             | Advanced                | Advanced exploitation                    | 🔜 Coming Soon              |

---

## Repository Structure

```
overthewire/
│
├── README.md               ← You are here
│
└── bandit/
    ├── README.md           ← Bandit index and level table
    └── walkthroughs/
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

Later wargames (Narnia, Behemoth, Vortex) involve binary exploitation and will require additional tools like `gdb` and Python. Those are covered in their respective walkthroughs.

---

## Useful Resources

| Resource | What it's for |
| -------- | ------------- |
| [explainshell.com](https://explainshell.com/) | Break down any shell command flag by flag |
| [tldr.sh](https://tldr.sh/) | Practical, example-driven man pages |
| [The Linux Command Line (free)](https://linuxcommand.org/tlcl.php) | Best beginner book for the terminal |
| [OverTheWire official site](https://overthewire.org/wargames/) | Game descriptions and hints |

---
