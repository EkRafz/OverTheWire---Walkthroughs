> **Level Goal:** Find the password for the next level on this page, but rightclicking has been blocked!

---

## Connection Details

| Field    | Value                                        |
| -------- | -------------------------------------------- |
| URL      | `http://natas1.natas.labs.overthewire.org`   |
| Username | `natas1`                                     |
| Password | *found in [Natas Level 0](natas-level-0.md)* |

---

## Tools Used

- A web browser
- **View Page Source** — read the raw HTML the server sent, before the browser renders it

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas1.natas.labs.overthewire.org` and log in with the password from Level 0.

The page displays:

```
You can find the password for the next level on this page, but rightclicking has been blocked!
```

Right-clicking is disabled via JavaScript, which blocks the context menu that normally contains _View Page Source_. The solution is the same as last level — the right-click menu was just one of several ways to get there.

---

### Understand the Restriction

Disabling right-click is a **client-side** control. It runs in your browser, on your machine, using JavaScript that the server sent you. That means:

- It can only block things the browser handles through that specific event.
- It has no effect on anything that bypasses the browser's context menu entirely.
- You already have the page's HTML — your browser downloaded it to render the page. The restriction only prevents one particular way of viewing it.

Client-side controls like this are not a security measure. They are a minor inconvenience at best, trivially bypassed, and a useful reminder that **anything the browser can render, you can read**.

---

### Step 2 — View the Page Source

Right-click is blocked, but the keyboard shortcut and address bar prefix are unaffected:

|Method|How|
|---|---|
|Keyboard shortcut|`Ctrl + U` (Windows/Linux), `⌘ + U` (Mac)|
|Address bar|Prefix the URL with `view-source:`|

Press `Ctrl + U`. The raw HTML opens in a new tab. Find the comment:

```html
<!--The password for natas2 is <password>-->
```

That is the password for [Natas Level 1 → 2](natas-level-1-level-2.md).

---

## Key Takeaways

- **Client-side restrictions only control what the browser does with a specific interaction.** Blocking right-click blocks one menu — it does not block `Ctrl + U`, `view-source:`, DevTools, `curl`, or any other way of reading the page.
- **The browser already has the full HTML** the moment the page loads. Any restriction that runs after that point cannot hide what was already delivered.
- This is a recurring theme in web security: controls enforced on the client are always bypassable by the client. Meaningful security lives on the server.

---