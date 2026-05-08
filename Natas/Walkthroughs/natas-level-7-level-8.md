> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| URL      | `http://natas8.natas.labs.overthewire.org`               |
| Username | `natas8`                                                 |
| Password | *found in [Natas Level 6 → 7](natas-level-6-level-7.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page provides a direct link to its own PHP source
- **PHP** (or any scripting environment) — to reverse the encoding pipeline

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas8.natas.labs.overthewire.org` and log in with the password from Level 7.

The page looks similar to Level 6: an input field asking for a secret, and a **View sourcecode** link. Click it.

---

### Step 2 — Read the Source Code

The relevant PHP:

```php
<?php

$encodedSecret = "XXXXXXXXXXXXXXXXXX";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
        print "Access granted. The password for natas9 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>
```

Two things are exposed:

- The **encoded secret**: `3d3d516343746d4d6d6c315669563362`
- The **encoding function**: `encodeSecret()`, which transforms a plaintext secret by passing it through three operations in sequence

The comparison runs your input through `encodeSecret()` and checks whether the result matches `$encodedSecret`. To pass, you need to supply the plaintext that produces that encoded value — which means reversing the encoding pipeline.

---

### Understand the Encoding Pipeline

`encodeSecret()` applies three transformations in order:

```
plain → base64_encode → strrev → bin2hex → encoded
```

**`base64_encode`** converts arbitrary binary data into a string of printable ASCII characters using a 64-character alphabet (`A–Z`, `a–z`, `0–9`, `+`, `/`). It is an encoding scheme, not encryption — it is fully reversible with `base64_decode` and carries no key or secret.

**`strrev`** reverses a string character by character. `"hello"` becomes `"olleh"`. Its inverse is itself — `strrev` applied twice returns the original.

**`bin2hex`** converts each byte of a string into its two-character hexadecimal representation. `"AB"` becomes `"4142"`. Its inverse is `hex2bin`.

None of these are cryptographic operations. They obscure the secret visually, but any attacker who can read the source code — as you can here — has everything needed to reverse the process.

---

### Step 3 — Reverse the Pipeline

To recover the plaintext, apply the inverse of each step in reverse order:

```
encoded → hex2bin → strrev → base64_decode → plain
```

Run it with `php -r`:

```bash
php -r 'echo base64_decode(strrev(hex2bin("XXXXXXXXXXXXX")));'
```

Output:

```
XXXXXX
```

That is the plaintext secret.

Alternatively, use [CyberChef](https://gchq.github.io/CyberChef/): chain **From Hex** → **Reverse** → **From Base64**, paste the encoded secret as input, and read the result.

---

### Step 4 — Submit the Secret

Go back to `http://natas8.natas.labs.overthewire.org`, enter `oubWYf2kBq` in the input field, and submit.

The page returns:

```
Access granted. The password for natas9 is <password>
```

That is the password for [Natas Level 8 → 9](natas-level-8-level-9.md).

---

## Key Takeaways

- **Encoding is not encryption.** Base64, hex, and string reversal are all fully reversible without any key. Storing a secret encoded this way and exposing the encoding function in source code is equivalent to storing it in plain text — anyone who reads the source recovers the secret immediately.
- **Exposed source code exposes the algorithm.** Security through obscurity only works while the algorithm stays hidden. The moment the encoding function is readable — as it is here via the sourcecode link — the protection collapses entirely.
- **Reversing a pipeline means inverting each step in reverse order.** If encoding is `A → B → C`, decoding is `C⁻¹ → B⁻¹ → A⁻¹`. Identifying each function and its inverse is the entire task.
- **CyberChef is the practical tool for encoding puzzles.** For chains of common encodings (hex, base64, URL encoding, etc.), CyberChef's drag-and-drop pipeline is faster than writing code. Worth knowing for any CTF involving encoded data.

---