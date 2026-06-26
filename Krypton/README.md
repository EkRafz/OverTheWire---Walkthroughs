```
		██╗  ██╗██████╗ ██╗   ██╗██████╗ ████████╗ ██████╗ ███╗   ██╗
		██║ ██╔╝██╔══██╗╚██╗ ██╔╝██╔══██╗╚══██╔══╝██╔═══██╗████╗  ██║
		█████╔╝ ██████╔╝ ╚████╔╝ ██████╔╝   ██║   ██║   ██║██╔██╗ ██║
		██╔═██╗ ██╔══██╗  ╚██╔╝  ██╔═══╝    ██║   ██║   ██║██║╚██╗██║
		██║  ██╗██║  ██║   ██║   ██║        ██║   ╚██████╔╝██║ ╚████║
		╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝   ╚═╝        ╚═╝    ╚═════╝ ╚═╝  ╚═══╝
```

---

## What is Krypton?

Krypton teaches the basics of **cryptography and cryptanalysis**. Unlike Bandit, there's no filesystem to explore, each level hands you ciphertext (and sometimes a binary) and the goal is always the same: **recover the password hidden inside it**. The levels move from classical, pen-and-paper ciphers toward attacks that require scripting and byte-level analysis.

> Like the Bandit and Natas walkthroughs, this one sticks to standard tools: `tr`, `base64`, `python`, `hexdump`/`xxd`, and manual frequency analysis. The game's own README suggests bringing in `cryptool` once you reach the later levels, this walkthrough notes where that happens but solves everything by hand first.

---

## Quick Start

Level 0 lives on the web, not SSH. Decode the Base64 string on the level page to get the password for `krypton1`:

```
https://overthewire.org/wargames/krypton/krypton0.html
```

From there:

```bash
ssh krypton1@krypton.labs.overthewire.org -p 2231
```

|Field|Value|
|---|---|
|Host|`krypton.labs.overthewire.org`|
|Port|`2231`|
|Username|`krypton1`|
|Password|_(decoded from Level 0)_|

Each subsequent level follows the pattern `kryptonX` as the username. Log in with the password you found on the previous level.

---

## Levels

| Level                                                          | Skills Introduced | Status |
| -------------------------------------------------------------- | ----------------- | ------ |
| [Level 0 → 1](Krypton/Walkthroughs/krypton-level-0-level-1.md) |                   | ✅      |
| [Level 1 → 2](Krypton/Walkthroughs/krypton-level-1-level-2.md) |                   | ✅      |
| [Level 2 → 3](Krypton/Walkthroughs/krypton-level-2-level-3.md) |                   | ✅      |
| [Level 3 → 4](Krypton/Walkthroughs/krypton-level-3-level-4.md) |                   | ✅      |
| [Level 4 → 5](Krypton/Walkthroughs/krypton-level-4-level-5.md) |                   | ✅      |
| [Level 5 → 6](Krypton/Walkthroughs/krypton-level-5-level-6.md) |                   | ✅      |
| [Level 6 → 7](Krypton/Walkthroughs/krypton-level-6-level-7.md) |                   | ✅      |

---

## Structure

```
Krypton/
│
├── krypton-README.md               ← You are here
└── Walkthroughs/
    ├── krypton-level-0.md
    ├── krypton-level-0-level-1.md
    ├── krypton-level-1-level-2.md
    └── ...
```

---

## Key Differences from Bandit and Natas

|Bandit|Natas|Krypton|
|---|---|---|
|SSH, Linux filesystem basics|HTTP, server-side web security|SSH, classical and applied cryptanalysis|
|Tools: `ls`, `cat`, `grep`…|Tools: browser DevTools, `curl`…|Tools: `tr`, `base64`, `python`, `hexdump`…|
|Teaches shell fundamentals|Teaches web vulnerabilities|Teaches cipher-breaking and cryptanalysis|
|No programming needed|No programming needed|Light scripting needed from level 3 onward|

---

## Core Tools You Will Use

| Tool                                 | What It Does                                                                     |
| ------------------------------------ | -------------------------------------------------------------------------------- |
| `base64`                             | Decodes/encodes Level 0 and other Base64-wrapped data                            |
| `tr`                                 | Character substitution — the workhorse for Caesar/ROT and monoalphabetic ciphers |
| `python` (or any scripting language) | Automating Caesar/Vigenère encryption, decryption, and brute-forcing             |
| `hexdump` / `xxd`                    | Byte-level inspection once ciphertext stops being plain ASCII                    |
| `cryptool`                           | GUI cryptanalysis tool the game itself recommends once levels get harder         |

---