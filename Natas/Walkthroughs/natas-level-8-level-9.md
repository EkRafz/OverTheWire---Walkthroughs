> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| URL      | `http://natas9.natas.labs.overthewire.org`               |
| Username | `natas9`                                                 |
| Password | *found in [Natas Level 7 → 8](natas-level-7-level-8.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **URL manipulation** — crafting the `?needle=` parameter to inject shell commands

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas9.natas.labs.overthewire.org` and log in with the password from Level 8.

The page displays a search box labelled "Find words containing:" and a submit button. Entering a word like `test` returns matching lines from what appears to be a dictionary file. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

The relevant PHP:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

Your input is placed directly into a shell command string with no sanitisation, then executed with `passthru()`. The full command the server runs looks like this when you search for `test`:

```bash
grep -i test dictionary.txt
```

There is nothing between your input and the shell. Whatever you type becomes part of the command.

---

### Understand Command Injection

**`passthru()`** is a PHP function that executes a shell command and sends its raw output directly to the browser. It is equivalent to running the command in a terminal on the server.

**Command injection** occurs when user input is embedded in a shell command without sanitisation. Shell interpreters recognise several characters that chain or terminate commands:

|Character|Effect|
|---|---|
|`;`|Run the next command regardless of whether the previous one succeeded|
|`&&`|Run the next command only if the previous one succeeded|
|`\|`|Pipe the output of one command into the next|

When the server builds the string `grep -i $key dictionary.txt` and your input is `test; cat /etc/passwd`, the shell sees:

```bash
grep -i test dictionary.txt; cat /etc/passwd
```

Two separate commands. The shell runs both. The server has no way to distinguish this from a legitimate search query — it assembled the string itself and handed it to the shell.

The root cause is a trust failure: the application assumed `$key` would always be a plain word, and built a shell command around that assumption. One semicolon breaks it entirely.

---

### Step 3 — Verify Code Execution

Before reading the password file, confirm that arbitrary commands execute. Enter this in the search box:

```
; whoami ;
```

The server runs:

```bash
grep -i  dictionary.txt; whoami ;
```

The `grep` part fails silently (empty pattern), but `whoami` executes and the page outputs:

```
natas9
```

The web server process is running as `natas9`, which has read permission on `/etc/natas_webpass/natas10`.

---

### Step 4 — Read the Password File

Enter this in the search box:

```
; cat /etc/natas_webpass/natas10 ;
```

The server runs:

```bash
grep -i  dictionary.txt; cat /etc/natas_webpass/natas10 ;
```

The page outputs the password for natas10.

The trailing `;` after the file path terminates the command cleanly, preventing the shell from trying to interpret anything else as an argument.

That is the password for [Natas Level 9 → 10](natas-level-9-level-10.md).

---

## Key Takeaways

- **Never pass user input directly into a shell command.** The moment unsanitised input reaches a function like `passthru()`, `exec()`, `system()`, or `shell_exec()`, the attacker controls the shell. There is no partial safety — even one unescaped `;` is enough.
- **`;` is the simplest command injection payload.** It terminates the intended command and starts a new one. If the application does not strip or escape shell metacharacters, `;` will almost always work.
- **Verify execution before reading sensitive files.** `whoami` is a harmless first payload that confirms you have code execution and tells you which user the server is running as — both useful facts before going further.
- **The fix is to avoid the shell entirely.** PHP has functions like `preg_grep()` that search arrays without ever invoking a shell. If a shell command is genuinely necessary, use `escapeshellarg()` to sanitise each argument, or run the command through a parameterised interface that does not interpret metacharacters.

---