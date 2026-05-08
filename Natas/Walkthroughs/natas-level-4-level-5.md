> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                                    |
| -------- | -------------------------------------------------------- |
| URL      | `http://natas5.natas.labs.overthewire.org`               |
| Username | `natas5`                                                 |
| Password | *found in [Natas Level 3 → 4](natas-level-3-level-4.md)* |

---

## Tools Used

- A web browser
- **Browser DevTools** — inspect and edit cookies stored by the page
- **`curl`** — a command-line tool for making HTTP requests with custom headers and cookies

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas5.natas.labs.overthewire.org` and log in with the password from Level 4.

The page displays:

```
Access disallowed. You are not logged in
```

The server thinks you are not logged in. There is no login form — so the login state must be tracked some other way. The obvious candidate is a cookie.

---

### Understand HTTP Cookies

HTTP is a **stateless** protocol. Each request is independent — the server has no built-in memory of previous requests. Cookies are the mechanism that works around this limitation.

When a server wants to associate state with a client — a session ID, a preference, a login flag — it sends a `Set-Cookie` header in its response. The browser stores that cookie and automatically includes it in every subsequent request to the same domain via a `Cookie` header.

A `Set-Cookie` header looks like this:

```
Set-Cookie: loggedin=0; Path=/
```

And the browser echoes it back in future requests:

```
Cookie: loggedin=0
```

The critical point: **cookies are stored on the client and sent by the client**. The server trusts what the client sends. If the cookie is a plain value — a flag, a username, an access level — the client can change it to anything.

This is not a flaw in the cookie mechanism itself. Cookies are correctly used for session tokens that the server can validate. The flaw is using a cookie to store a trust decision — like "is this user logged in?" — as a raw value the client can freely edit.

---

### Step 2 — Inspect the Cookie

Open browser DevTools with `F12` and navigate to the **Storage** tab (Firefox) or **Application → Cookies** (Chrome/Edge).

Under the cookies for `natas5.natas.labs.overthewire.org`, find:

```
Name:  loggedin
Value: 0
```

The server is reading this cookie to determine access. A value of `0` means not logged in.

---

### Step 3 — Edit the Cookie

Double-click the value field in DevTools and change `0` to `1`. Refresh the page.

The server reads the updated cookie and returns:

```
Access granted. The password for natas6 is <password>
```

That is the password for [Natas Level 5 → 6](natas-level-5-level-6.md).

---

### Alternative — curl

You can also set the cookie directly in the request without touching a browser at all:

```bash
curl -u natas5:<your-password> \
     -H "Cookie: loggedin=1" \
     http://natas5.natas.labs.overthewire.org/
```

The `-H "Cookie: ..."` flag injects the cookie header manually, bypassing whatever value the server originally set.

---

## Key Takeaways

- **Cookies are client-controlled.** The server sets a cookie, but the client stores it and sends it back. Any plain-value cookie — a flag, a role, an ID — can be read and modified by the client.
- **Storing trust decisions in cookies is dangerous.** A `loggedin=0/1` flag is the most obvious example: the client decides their own login state. Session tokens avoid this by storing a random opaque value the server validates against its own records — the token itself carries no editable meaning.
- **DevTools is a first-class security tool.** The Storage tab gives you direct read/write access to every cookie on the page. No proxy required, no extra software.
- **Cookies are just headers.** `curl -H "Cookie: ..."` sets them as freely as any other header. Anything the browser sends, you can send manually.

---