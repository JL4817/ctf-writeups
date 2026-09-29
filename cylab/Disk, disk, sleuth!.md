# Disk, Disk, Sleuth!

## Challenge Overview

The challenge provides a **compressed disk image** (`dds1-alpine.flag.img.gz`) and asks us to find a hidden flag inside it using `srch_strings` from The Sleuth Kit.

---

## Exploitation

### Step 1 — Decompress the Disk Image

```bash
gunzip dds1-alpine.flag.img.gz
```

### Step 2 — Search for the Flag

`srch_strings` extracts all printable strings from a disk image at the raw byte level — including strings in unallocated space that normal file system tools would miss. We pipe the output directly into `grep` to filter for the flag format:

```bash
srch_strings dds1-alpine.flag.img | grep 'academy{'
```

```
SAY academy{f0r3ns1c4t0r_n30phyt3_6502313d}
```

🏁 Flag: `academy{f0r3ns1c4t0r_n30phyt3_6502313d}`

---

## Key Takeaway

> **`srch_strings` searches raw disk bytes**, not just active files. Data deleted from a file system often remains physically present on disk until overwritten — making string searches against raw images a powerful forensic technique for recovering hidden or deleted content in a single command.
```
