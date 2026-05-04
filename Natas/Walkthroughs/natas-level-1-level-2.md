> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                              |
| -------- | -------------------------------------------------- |
| URL      | `http://natas2.natas.labs.overthewire.org`         |
| Username | `natas2`                                           |
| Password | *found in [Natas Level 0 → 1](natas-level-0-level-1.md)* |

---

## Tools Used

- A web browser
- **View Page Source** — read the raw HTML the server sent
- **Directory listing** — browsing a server directory directly in the browser

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas2.natas.labs.overthewire.org` and log in with the password from Level 1.

The page displays:

```
There is nothing on this page.
```

View the source with `Ctrl + U`. This time there is no comment containing the password, but there is an `<img>` tag:

```html
<img src="files/pixel.png">
```

The page loads an image from a subdirectory called `files/`. That directory exists on the server.

---

### Understand Directory Listing

Web servers store files in directories, just like your local filesystem. When you request a file by URL, the server returns that file. But if you request a **directory** URL and the server has not placed an `index.html` inside it, many servers will instead return a **directory listing** — a plain page showing every file in that folder.

This is a misconfiguration. A directory listing exposes the full contents of a folder to anyone who knows — or guesses — the path. Developers often forget that uploading an image or a script also makes the entire containing directory browsable.

---

### Step 2 — Browse the Directory

Remove `pixel.png` from the URL and navigate directly to the directory:

```
http://natas2.natas.labs.overthewire.org/files/
```

The server returns a directory listing:

```
Index of /files
pixel.png
users.txt
```

`users.txt` should not be publicly accessible — but it is.

---

### Step 3 — Open users.txt

Navigate to:

```
http://natas2.natas.labs.overthewire.org/files/users.txt
```

The file contains a list of usernames and passwords. Find the entry for `natas3`:

```
natas3:<password>
```

That is the password for [Natas Level 2 → 3](natas-level-2-level-3.md).

---

## Key Takeaways

- **Page source reveals more than just the visible content.** Referenced assets — images, scripts, stylesheets — point to paths on the server worth exploring.
- **Directory listing is a misconfiguration**, not a feature. If a directory has no `index.html`, many servers expose its full contents by default. It should be explicitly disabled in production.
- **Every path in a URL is a potential attack surface.** If you can reach `files/pixel.png`, you can try `files/` — the server decides what to show, and misconfigured servers show everything.

---