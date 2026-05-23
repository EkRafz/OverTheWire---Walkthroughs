> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas24.natas.labs.overthewire.org`                  |
| Username | `natas24`                                                    |
| Password | *found in [Natas Level 22 → 23](natas-level-22-level-23.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **`curl`** — to submit an array via GET parameter

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas24.natas.labs.overthewire.org` and log in with the password from Level 23.

The page looks identical to Level 23 — a single password field:

```
Password: [          ] [Login]
```

There is also a **View sourcecode** link. Click it.

---

### Step 2 — Read the Source Code

```php
<?php
if(array_key_exists("passwd",$_REQUEST)){
    if(!strcmp($_REQUEST["passwd"],"<censored>")){
        echo "<br>The credentials for the next level are:<br>";
        echo "<pre>Username: natas25 Password: <censored></pre>";
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

Same entry gate as Level 23 — runs only if `passwd` is present in the request.

```php
if(!strcmp($_REQUEST["passwd"],"<censored>")){
```

`strcmp($a, $b)` compares two strings lexicographically. It returns:

- a negative integer if `$a` is less than `$b`
- a positive integer if `$a` is greater than `$b`
- `0` if the two strings are identical

The condition is `!strcmp(...)`, the logical NOT of the return value. In PHP, `!0` is `true`, and `!<any non-zero integer>` is `false`. So the password block executes only when `strcmp()` returns exactly `0`, only when the two arguments are identical strings.

The actual password is censored in the source. There is no injection point, no bypass via type coercion like Level 23, and no side channel. The comparison looks airtight: `strcmp()` is a well-defined C-backed function that compares strings. Guessing or brute-forcing the password is not realistic.

The vulnerability is not in the logic. It is in `strcmp()` itself — specifically, in how PHP handles it when the first argument is not a string.

---

### Understand the `strcmp()` Array Vulnerability

`strcmp()` is documented to accept two strings. PHP is a loosely typed language and does not enforce argument types at the call site. If you pass a non-string as the first argument, PHP does not reject the call — it attempts to coerce the value and proceeds.

In PHP versions prior to 7, passing an array as the first argument to `strcmp()` triggers an internal error: the function cannot compare an array to a string. Rather than throwing a fatal exception, PHP emits a warning and returns `NULL`.

The condition is `!strcmp(...)`. `!NULL` evaluates to `true` in PHP — `NULL` is falsy, and negating a falsy value produces `true`. The check passes.

The full chain:

```
strcmp(array, "secret") → NULL   (type mismatch, emits warning)
!NULL                   → true   (NULL is falsy; !falsy = true)
password block executes
```

The password is never compared. The function silently fails in a way that the surrounding code interprets as a match.

**How to pass an array via HTTP:** PHP's query string parser treats parameter names ending in `[]` as arrays. `passwd[]=anything` causes `$_REQUEST["passwd"]` to be the array `["anything"]` rather than the string `"anything"`. The value inside the brackets does not matter — any value, or no value at all, produces the array.

This behaviour is specific to older PHP. From PHP 7 onward, `strcmp()` with a non-string argument throws a fatal `TypeError` and the script halts — the same payload crashes the page instead of bypassing the check. The natas server runs a PHP version where the old behaviour applies.

---

### Step 3 — Submit the Payload

#### Via the browser URL bar

Navigate directly to:

```
http://natas24.natas.labs.overthewire.org/?passwd[]=anything
```

The `[]` suffix on `passwd` tells PHP's query string parser to treat the value as an array element. The browser may URL-encode the brackets as `%5B%5D` — both forms work.

The page returns a PHP warning followed by the password:

```
**Warning**: strcmp() expects parameter 1 to be string, array given in **/var/www/natas/natas24/index.php** on line **23**

The credentials for the next level are:
Username: natas25
Password: <password>
```

The warning confirms exactly what happened: `strcmp()` received an array, returned `NULL`, and the `!NULL` condition passed.

#### Via `curl`

```bash
curl -u natas24:<password> \
     "http://natas24.natas.labs.overthewire.org/?passwd[]=anything"
```

|Flag / argument|Meaning|
|---|---|
|`-u natas24:<password>`|HTTP Basic Authentication|
|`?passwd[]=anything`|Submits `passwd` as a one-element array — the value `"anything"` is irrelevant|

The response contains the same warning and the password for the next level.

That is the password for [Natas Level 24 → 25](natas-level-24-level-25.md).

---

## Key Takeaways

- **`strcmp()` returns `NULL` on a type mismatch in older PHP, not an error.** When given a non-string first argument, PHP emits a warning but continues execution and returns `NULL`. The function's documented contract assumes strings; the runtime does not enforce it.
- **`!NULL` is `true` in PHP.** `NULL` is falsy. The `!` operator negates falsy values to `true`. A guard written as `!strcmp(...)` passes when `strcmp()` returns `0` (strings match) — but also when it returns `NULL` (type error). The two cases are indistinguishable to the condition.
- **`param[]=value` in a query string submits an array.** PHP's query string parser treats the `[]` suffix as an instruction to build an array. Any function that expects a string and receives `$_REQUEST["param"]` without a type check is exposed to this technique.
- **Type checks on user input are not optional.** The correct fix is to verify that `$_REQUEST["passwd"]` is a string before passing it to `strcmp()` — or to use strict comparison (`===`) instead, which would not call `strcmp()` at all and would not be fooled by a type mismatch. Relying on a library function's documented behaviour without validating that its inputs meet its documented preconditions is a structural mistake.
- **This vulnerability is version-dependent.** PHP 7 changed the behaviour: passing a non-string to `strcmp()` now throws a fatal `TypeError` rather than returning `NULL`. The same payload crashes a PHP 7 application instead of bypassing it. Code that was exploitable on PHP 5 may behave differently — but not necessarily safely — under newer versions.

---