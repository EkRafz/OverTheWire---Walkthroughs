> **Level Goal:** Find the password for the next level.

---

## Connection Details

| Field    | Value                                              |
| -------- | -------------------------------------------------- |
| URL      | `http://natas4.natas.labs.overthewire.org`         |
| Username | `natas4`                                           |
| Password | *found in [Natas Level 2 → 3](natas-level-2-level-3.md)* |

---

## Tools Used

- A web browser
- **`curl`** — a command-line tool for making HTTP requests
- **HTTP `Referer` header** — a request header that tells the server which page you came from

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas4.natas.labs.overthewire.org` and log in with the password from Level 3.

The page displays:

```
Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
```

The server is checking where your request came from and rejecting it. To get access, the request needs to appear to come from `http://natas5.natas.labs.overthewire.org/`.

---

### Understand the Referrer Header

When you click a link or submit a form, your browser automatically includes an HTTP **`Referer`** header in the request. It contains the URL of the page you were on when you made the request, telling the server where you "came from."

A raw HTTP request with a `Referrer` header looks like this:

```
GET / HTTP/1.1
Host: natas4.natas.labs.overthewire.org
Referrer: http://natas5.natas.labs.overthewire.org/
```

Servers sometimes use `Referrer` as an access control, only serving content if the request appears to come from a trusted page. This is not a reliable security mechanism because:

- The `Referer` header is sent by the client, so the client can set it to anything.
- It is trivially forged with `curl`, a proxy, or browser DevTools.

---

### Understand curl

`curl` is a command-line tool for transferring data with URLs. When you need to craft an HTTP request with specific headers, authentication, or methods — anything a browser does automatically and silently — `curl` lets you control every detail explicitly.

The relevant flags for this level come directly from the help output (`curl --help all`):

```
-u, --user <user:password>    Server user and password
-H, --header <header/@file>   Pass custom header(s) to server
-e, --referer <URL>           Referrer URL
```

**`-u, --user <user:password>`** Sends HTTP Basic Authentication credentials with the request. The browser normally handles this through the login popup — `-u` lets you supply credentials directly on the command line without any browser interaction.

**`-H, --header <header/@file>`** Injects an arbitrary header into the request. The format is `"Header-Name: value"`. You can use `-H` multiple times in a single command to add as many headers as needed. This is the general-purpose flag for any header the browser would normally set automatically.

**`-e, --referer <URL>`** A dedicated shorthand for setting the `Referer` header specifically. Equivalent to `-H "Referer: <URL>"` — both work, `-e` is just more readable when `Referer` is all you need.

---

### Step 2 — Forge the Referer Header with curl

`curl` lets you set any HTTP header manually with the `-H` flag. The `-u` flag handles HTTP Basic Auth.

```bash
curl -u natas4:<your-password> \
     -H "Referer: http://natas5.natas.labs.overthewire.org/" \
     http://natas4.natas.labs.overthewire.org/
```

The server receives the request, sees the expected `Referer`, and returns the page HTML. Look for the password in the output:

```
Access granted. The password for natas5 is <password>
```

That is the password for [Natas Level 4 → 5](natas-level-4-level-5.md).

---

## Key Takeaways

- **The `Referer` header is client-controlled.** Any header sent by the browser can be set to an arbitrary value by the client. A server that uses it for access control is trusting input it cannot verify.
- **`curl -H`** lets you inject any HTTP header into a request. This makes `curl` an essential tool any time a browser's automatic behaviour gets in the way — you can construct exactly the request you want, header by header.
- **HTTP headers are part of the attack surface.** `Referer` is one of many headers servers read and act on. `User-Agent`, `X-Forwarded-For`, `Cookie`, and `Content-Type` are others worth manipulating when a server's behaviour seems to depend on them.

---