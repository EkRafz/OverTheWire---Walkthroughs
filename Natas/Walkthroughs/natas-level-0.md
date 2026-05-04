> **Level Goal:** Find the password for the next level on this page.

---

## Connection Details

|Field|Value|
|---|---|
|URL|`http://natas0.natas.labs.overthewire.org`|
|Username|`natas0`|
|Password|`natas0`|

---

## Tools Used

- A web browser
- **View Page Source** — read the raw HTML the server sent, before the browser renders it

---

## Solution

### Step 1 — Open the URL

Navigate to `http://natas0.natas.labs.overthewire.org` in your browser. You will be prompted for credentials — enter `natas0` / `natas0`.

The page displays a single line:

```
You can find the password for the next level on this page.
```

Nothing else is visible. The password is not rendered on screen, which means it is somewhere in the page's source.

---

### Understand View Page Source

Every web page is built from **HTML** — a plain-text document the server sends to your browser. The browser reads that HTML and renders it visually, but in doing so it hides a lot of detail: comments, hidden elements, metadata, and anything the developer did not intend to display.

>**View Page Source** lets you read the raw HTML exactly as the server delivered it, before any rendering takes place. Developers sometimes leave sensitive information in HTML comments or hidden elements that are invisible in the normal browser view but fully present in the source.

|Method|How|
|---|---|
|Keyboard shortcut|`Ctrl + U` (Windows/Linux), `⌘ + U` (Mac)|
|Right-click menu|Right-click anywhere → _View Page Source_|
|Address bar|Prefix the URL with `view-source:`|

---

### Step 2 — View the Page Source

Press `Ctrl + U`. A new tab opens showing the raw HTML of the page.

Look for an HTML comment — comments are written between `<!--` and `-->` and are never rendered by the browser:

```html
<!--The password for natas1 is <password>-->
```

That is the password for [Natas Level 0 → 1](natas-level-0-level-1.md).

---

## Key Takeaways

- **What the browser renders and what the server sent are not the same thing.** View Page Source is always your first move when a page claims to have information you cannot see.
- **HTML comments** (`<!-- ... -->`) are invisible to normal users but fully readable in source. They are a common place for developers to leave notes — or accidentally leave credentials.
- This technique scales: even as Natas gets harder, reading source remains a fundamental first step before reaching for any other tool.

---
