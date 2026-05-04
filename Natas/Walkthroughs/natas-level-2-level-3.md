> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                              |
| -------- | -------------------------------------------------- |
| URL      | `http://natas3.natas.labs.overthewire.org`         |
| Username | `natas3`                                           |
| Password | *found in [Natas Level 1 → 2](natas-level-1-level-2.md)* |

---

## Tools Used

- A web browser
- **View Page Source** — read the raw HTML the server sent
- **`robots.txt`** — a publicly accessible file that lists paths the server wants crawlers to ignore

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas3.natas.labs.overthewire.org` and log in with the password from Level 2.

The page displays:

```
There is nothing on this page.
```

View the source with `Ctrl + U`. No password, no referenced files — but there is a comment:

```html
<!-- No more information leaks!! Not even Google will find it this time... -->
```

The mention of Google is the hint. Google discovers pages using automated crawlers — and there is a standard mechanism to tell those crawlers where not to go.

---

### Understand robots.txt

**`robots.txt`** is a plain-text file placed at the root of a web server. It follows the **Robots Exclusion Protocol**, a convention that tells web crawlers which paths they should not visit or index.

A typical `robots.txt` looks like this:

```
User-agent: *
Disallow: /private/
```

`User-agent: *` applies the rule to all crawlers. `Disallow` lists paths the server owner does not want indexed.

Two critical points:

- **`robots.txt` is always publicly accessible** at `/<domain>/robots.txt`. It has to be — crawlers need to read it. Any human can read it too.
- **`robots.txt` is not a security control.** It is a polite request. Crawlers that follow it will skip the listed paths, but nothing prevents a person — or a malicious crawler — from visiting them directly.

In practice, `Disallow` entries in `robots.txt` are a map of paths the server owner considered sensitive enough to hide from search engines. That makes it a useful reconnaissance resource.

---

### Step 2 — Read robots.txt

Navigate to:

```
http://natas3.natas.labs.overthewire.org/robots.txt
```

The file contains:

```
User-agent: *
Disallow: /s3cr3t/
```

The server is telling crawlers to stay out of `/s3cr3t/`. That is exactly where to look.

---

### Step 3 — Browse the Directory

Navigate to:

```
http://natas3.natas.labs.overthewire.org/s3cr3t/
```

Directory listing is enabled, revealing:

```
Index of /s3cr3t
users.txt
```

---

### Step 4 — Open users.txt

Navigate to:

```
http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt
```

Find the entry for `natas4`:

```
natas4:<password>
```

That is the password for [Natas Level 3 → 4](natas-level-3-level-4.md).

---

## Key Takeaways

- **`robots.txt` is a reconnaissance resource, not a security control.** Any path listed under `Disallow` is something the server owner wanted hidden from search engines — which makes it a list of interesting paths worth visiting manually.
- **Hiding a path does not protect it.** Security through obscurity fails the moment the obscuring mechanism is itself public. `robots.txt` must be readable by crawlers, which means it is readable by everyone.
- **Comments in source code leak intent.** The hint here was not technical — it was a developer note. Reading source carefully, including throwaway comments, often points to the next step.

---