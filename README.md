

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

| Wargame                             | Difficulty              | Topic                                                   | Status                     |
| ----------------------------------- | ----------------------- | ---------------------------------------------- | -------------------------- |
| [Bandit](./Bandit/bandit-README.md) | Beginne Linux fundamentals, SSH, shell basics and git  n  n  n  n  n  n  n  n  n  | 🟢 Completed (Levels 0–33) |
| Leviathan                           | Beginner                | Basic                                                   | 🔜 Coming Soon             |
| Natas                               | Beginner → Intermediat                                                            | 🔜 Coming Soon             |
| Krypton                             | Begin                                                                             | 🔜 Coming Soon             |
| Narnia                              | Intermediate                      C bi  ary e  ploit  tion,  memor   corr  ption  | 🔜 Coming Soon             |
| Behemoth                                                                                                                | 🔜 Coming Soon             |
| Utumno                                                                                                                  | 🔜 Coming Soon             |
| Ma                                                                                                                      | 🔜 Coming Soon             |
| V                                                                                                                       | 🔜 Coming S                                                                                                                                            | 🔜 C                                                                                                                                                   |                                                                                                                                                        | 🔜 Coming Soon             |

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
