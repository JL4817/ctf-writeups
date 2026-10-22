## Challenge Overview

The challenge provides a file called `flag.txt`. Despite the `.txt` extension, it contains a hidden flag.

---

## Exploitation

### Step 1 — Inspect the File Header

Operating systems don't rely solely on file extensions to determine file type — they read the **magic bytes** at the start of the file. Opening `flag.txt` in a text editor or running `file` on it reveals:

```bash
file flag.txt
# flag.txt: PNG image data
```

The file starts with recognizable PNG magic bytes and headers:

```
‰PNG
IHDR
pHYs
IDAT
```

This is not a text file — it is a **PNG image** with a `.txt` extension.

### Step 2 — Rename and Open

Renaming the file to `flag.png` allows an image viewer to open it correctly, revealing the flag visually inside the image.

```bash
mv flag.txt flag.png
open flag.png
```

🏁 Flag retrieved.

---

## Key Takeaway

> **File extensions are just hints — they don't define the actual file type.** The real format is determined by the magic bytes at the start of the file. Tools like `file` on Linux read these bytes to identify the true format, regardless of what the extension says. Attackers often exploit this by disguising malicious files with innocent-looking extensions.
```
