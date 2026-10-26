# Disk, Disk, Sleuth! II

## Challenge Overview

The challenge provides a compressed disk image (`dds2-alpine.flag.img.gz`) containing a file named `down-at-the-bottom.txt`. The goal is to navigate the file system using Sleuth Kit tools to find and read it.

---

## Exploitation

### Step 1 — Decompress and Inspect the Partition Table

```bash
gunzip dds2-alpine.flag.img.gz
mmls dds2-alpine.flag.img
```

```
002:  000:000   0000002048   0000262143   0000260096   Linux (0x83)
```

The Linux partition starts at sector **2048** — this offset is used for all subsequent commands.

### Step 2 — Inspect the File System

```bash
fsstat -o 2048 dds2-alpine.flag.img
```

Confirms the file system is **Ext3**.

### Step 3 — List the Root Directory

```bash
fls -o 2048 dds2-alpine.flag.img
```

This shows the top-level directories. We spot `root` at inode **18290**, which is a likely location for the flag file.

### Step 4 — Look Inside the root Directory

```bash
fls -a -o 2048 dds2-alpine.flag.img 18290
```

```
r/r 18291:    down-at-the-bottom.txt
```

✅ The file is at inode **18291**.

### Step 5 — Read the File with icat

Since the file is inside a disk image (not mounted), we can't use `cat` directly. `icat` reads a file by its inode number:

```bash
icat -o 2048 dds2-alpine.flag.img 18291
```

The file renders the flag in decorative ASCII art — reading each letter in order:

```
academy{f0r3ns1c4t0r_n0v1c3_95c645ad}
```

🏁 Flag: `academy{f0r3ns1c4t0r_n0v1c3_95c645ad}`

---

## Key Takeaway

> **The Sleuth Kit (`mmls`, `fls`, `icat`) allows full file system navigation without mounting a disk image.** `fls` lists files by inode, and `icat` extracts file contents directly by inode number — essential skills when working with forensic disk images where mounting isn't possible or desirable.
```
