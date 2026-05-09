> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                      |
| -------- | ---------------------------------------------------------- |
| URL      | `http://natas11.natas.labs.overthewire.org`                |
| Username | `natas11`                                                  |
| Password | *found in [Natas Level 9 → 10](natas-level-9-level-10.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Browser DevTools** — to read and overwrite the `data` cookie
- **PHP** (or any scripting environment) — to recover the XOR key and forge the cookie

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas11.natas.labs.overthewire.org` and log in with the password from Level 10.

The page displays a background colour picker (defaulting to `#ffffff`) and the notice: **"Cookies are protected with XOR encryption."** Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';
    
    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }
    
    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
        $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
        if(is_array($tempdata) &&
           array_key_exists("showpassword", $tempdata) &&
           array_key_exists("bgcolor", $tempdata)) {
            if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
                $mydata['showpassword'] = $tempdata['showpassword'];
                $mydata['bgcolor'] = $tempdata['bgcolor'];
            }
        }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}

$data = loadData($defaultdata);

if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}

saveData($data);

?>

<? if($data["showpassword"] == "yes") {
    print "The password for natas12 is <censored><br>";
} ?>
```

There is more going on here than in the previous two levels. Break it down function by function before deciding how to attack it.

---

### Understand the Data Flow

The server uses a `data` cookie to persist state across requests. The cookie stores a PHP array with two fields:

```php
array("showpassword" => "no", "bgcolor" => "#ffffff")
```

**Writing the cookie** — `saveData()` serialises the array through three transformations in sequence:

```
array → json_encode → xor_encrypt → base64_encode → cookie string
```

**Reading the cookie** — `loadData()` reverses those steps:

```
cookie string → base64_decode → xor_encrypt → json_decode → array
```

The password is printed only if `$data["showpassword"] == "yes"`. The server sets that field itself from the cookie it receives. If you can forge a cookie whose decrypted content contains `"showpassword":"yes"`, the server will print the password.

---

### Understand XOR Encryption

The `xor_encrypt` function applies a bitwise XOR between each byte of its input and the corresponding byte of a repeating key:

```php
for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
}
```

The `%` (modulo) operator causes the key to wrap around and repeat when the input is longer than the key. A four-character key applied to a 52-character input simply repeats 13 times.

XOR has a fundamental algebraic property: it is its own inverse. If:

```
plaintext XOR key = ciphertext
```

Then:

```
ciphertext XOR key = plaintext
ciphertext XOR plaintext = key
```

This means that if you know any **two of the three values**, you can compute the third. The server gives you two of them:

- **Ciphertext**: the `data` cookie value (after base64-decoding), which you can read from your browser right now
- **Plaintext**: the default array `{"showpassword":"no","bgcolor":"#ffffff"}` — it is hardcoded in the source as `$defaultdata` and is what the server writes when no cookie is present

The key is hidden (`<censored>` in the source), but you can recover it by XORing the ciphertext against the known plaintext.

This is a **known-plaintext attack**: the ability to recover a key when you possess a matching plaintext/ciphertext pair.

---

### Step 3 — Read the Current Cookie

Open DevTools (`F12`), go to **Application → Cookies**, and find the `data` cookie. Its value will look something like:

```
HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg=
```

The `%3D` is just URL-encoding for `=`

---

### Step 4 — Recover the XOR Key

Write a short PHP script that XORs the base64-decoded cookie against the known default plaintext. Since `xor_encrypt(ciphertext, key) = plaintext`, and XOR is its own inverse, `xor_encrypt(ciphertext, plaintext) = key`.

```php
<?php
$cookie    = "HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg=";
$plaintext = json_encode(array("showpassword"=>"no", "bgcolor"=>"#ffffff"));
$cipher    = base64_decode($cookie);

$outText = '';
for($i = 0; $i < strlen($cipher); $i++) {
    $outText .= $cipher[$i] ^ $plaintext[$i % strlen($plaintext)];
}
echo $outText;
?>
```

Run it:

```bash
php natas11.php
```

Output:

```
eDWoeDWoeDWoeDWoeDWoeDWoeDWoeDWoeDWoeDWoe
```

The repeating pattern `eDWo` is the XOR key. The repetition confirms both that the key is shorter than the plaintext and that the decryption is correct — a real key repeating cleanly like this is unmistakeable.

---

### Step 5 — Forge the Malicious Cookie

Now that you have the key, reverse the `saveData` pipeline but with `"showpassword":"yes"`:

```
modified array → json_encode → xor_encrypt(key="eDWo") → base64_encode → forged cookie
```

```php
<?php
function xor_encrypt($in) {
    $key     = 'eDWo';
    $outText = '';
    for($i = 0; $i < strlen($in); $i++) {
        $outText .= $in[$i] ^ $key[$i % strlen($key)];
    }
    return $outText;
}

$data = array("showpassword" => "yes", "bgcolor" => "#ffffff");
echo base64_encode(xor_encrypt(json_encode($data)));
?>
```

Run it:

```bash
php natas11.php
```

Output:

```
HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5
```

This is the forged cookie value. The only difference from the original cookie is the bytes that encode `"showpassword":"yes"` instead of `"showpassword":"no"`.

---

### Step 6 — Inject the Cookie and Reload

In DevTools, navigate to **Application → Cookies → natas11**. Double-click the value of the `data` cookie and replace it with the forged value:

```
HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5
```

Reload the page. The server decrypts the cookie, finds `"showpassword":"yes"`, and prints:

```
The password for natas12 is <password>
```

That is the password for [Natas Level 11 → 12](natas-level-11-level-12.md).

---

## Key Takeaways

- **XOR with a short repeating key is not encryption.** A key shorter than the plaintext that repeats character-by-character is a stream cipher with near-zero security. Any attacker who recovers even a few bytes of plaintext can extract the full key and decrypt or forge any ciphertext.
- **Known-plaintext attacks break XOR trivially.** The algebraic identity `key = plaintext XOR ciphertext` means that if you know what any part of the plaintext is supposed to be — and here the entire default state is hardcoded in the source — you get the key for free.
- **Trusting client-supplied cookies to store server state is dangerous.** The server delegated the `showpassword` field to the client. Even "encrypted" client storage is only as secure as the encryption scheme, and here the scheme is broken. Sensitive state like access flags must be kept server-side (e.g. in a server session store), where the client cannot read or tamper with it.
- **The pipeline is the vulnerability, not just the key.** Even if the key had been stronger, exposing the encoding pipeline (`json_encode → xor_encrypt → base64_encode`) in source code gives attackers all the information they need to understand what a valid cookie looks like and craft one. Security by obscurity only holds while the algorithm stays hidden.

---
