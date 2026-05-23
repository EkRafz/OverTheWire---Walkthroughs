> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas23.natas.labs.overthewire.org`                  |
| Username | `natas23`                                                    |
| Password | *found in [Natas Level 21 → 22](natas-level-21-level-22.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas23.natas.labs.overthewire.org` and log in with the password from Level 22.

The page displays a single password field:

```
Password: [          ] [Login]
```

There is also a **View sourcecode** link. Click it.

---

### Step 2 — Read the Source Code

```php
<?php
if(array_key_exists("passwd",$_REQUEST)){
    if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
        echo "<br>The credentials for the next level are:<br>";
        echo "<pre>Username: natas24 Password: <censored></pre>";
    } else{
        echo "<br>Wrong!<br>";
    }
}
// morla / 10111
?>
```

Walking through it:

```php
if(array_key_exists("passwd",$_REQUEST)){
```

Runs only if a `passwd` parameter was submitted — either via GET or POST, since `$_REQUEST` captures both.

```php
if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
```

Two conditions joined by `&&` — both must be true:

1. `strstr($_REQUEST["passwd"], "iloveyou")` → the string `"iloveyou"` must appear somewhere inside the submitted value
2. `$_REQUEST["passwd"] > 10` → the submitted value must be greater than the integer `10`

At first glance these seem contradictory. A string like `"iloveyou"` contains no digits, so comparing it to `10` should fail. And a bare number like `"42"` satisfies the numeric check but contains no `"iloveyou"`. The question is: does PHP agree with that reading?

It does not.

---

### Understand PHP Type Juggling

PHP is a **loosely typed** language. Variables have no declared type; PHP infers and converts types automatically based on context. When the `>` operator compares a string to an integer, PHP does not refuse or error — it converts the string to a number first and then compares.

The rule PHP applies when converting a string to a number: read leading numeric characters from the string until the first non-numeric character, and use the resulting number. If the string starts with a digit, PHP extracts it. If it starts with a non-digit, the result is `0`.

Examples:

|String|PHP numeric conversion|`> 10`|
|---|---|---|
|`"iloveyou"`|`0`|false|
|`"10iloveyou"`|`10`|false|
|`"11iloveyou"`|`11`|**true**|
|`"100iloveyou"`|`100`|**true**|
|`"iloveyou11"`|`0`|false|

The leading digits determine the numeric value. Everything after the first non-digit character is discarded for the comparison. The string itself is unchanged — `strstr()` still sees the full original value including `"iloveyou"`.

This means both conditions can be satisfied simultaneously by a single string: put a number greater than `10` at the start, followed immediately by `"iloveyou"`.

**`strstr()` and containment:** `strstr($haystack, $needle)` returns the portion of `$haystack` from the first occurrence of `$needle` to the end, or `false` if not found. It does not require an exact match — it searches for the substring anywhere in the input. `"11iloveyou"` contains `"iloveyou"`, so `strstr()` returns `"iloveyou"`, which is truthy.

Both checks pass with `"11iloveyou"`:

- `strstr("11iloveyou", "iloveyou")` → `"iloveyou"` (truthy)
- `"11iloveyou" > 10` → PHP converts to `11`, `11 > 10` → true

---

### Step 3 — Submit the Payload

Enter `11iloveyou` in the password field and click Login. Any integer greater than `10` prepended to `iloveyou` works — `11iloveyou`, `42iloveyou`, `100iloveyou` are all equivalent.

The page returns:

```
The credentials for the next level are:
Username: natas24 Password: <password>
```

That is the password for [Natas Level 23 → 24](natas-level-23-level-24.md).

---

## Key Takeaways

- **PHP silently converts types during comparison.** When `>` compares a string against an integer, PHP converts the string to a number rather than throwing an error or returning false. The developer may expect a string to fail a numeric comparison — PHP does not share that expectation.
- **String-to-number conversion reads only the leading digits.** PHP extracts the longest leading numeric prefix from a string and uses that as the integer value. `"11iloveyou"` becomes `11`. `"iloveyou"` becomes `0`. The position of the digits matters: a number embedded in the middle or at the end contributes nothing.
- **`strstr()` checks containment, not equality.** It returns truthy if the needle appears anywhere inside the haystack. A developer who writes `strstr($input, "iloveyou")` expecting to enforce that the password _is_ `"iloveyou"` has a logic error — the check passes for any string that merely _contains_ it.
- **Two independently correct checks can combine into a broken constraint.** Neither the `strstr()` check nor the `> 10` check is wrong on its own. The flaw is in assuming they collectively enforce mutually exclusive requirements. PHP's type coercion makes them simultaneously satisfiable. Correct enforcement of an exact password requires `===` (strict equality, no type coercion) rather than `strstr()` and a numeric comparison.

---