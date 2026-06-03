# Sleuthkit Intro

## Challenge Overview

The challenge provides a **disk image** and a netcat service that asks for the size of the Linux partition in sectors. Answer correctly to receive the flag.

---

## Exploitation

### Step 1 — Analyse the Disk Image

We use `mmls` to list the partition table of the disk image:

```bash
mmls disk.img
```

```
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000204799   0000202752   Linux (0x83)
```

The Linux partition (`0x83`) has a length of **202752 sectors**.

### Step 2 — Submit the Answer

```bash
nc saturn.picoctf.net 55459
```

```
What is the size of the Linux partition in the given disk image?
Length in sectors: 202752
```

🏁 Flag retrieved.

---

## Key Takeaway

> **`mmls` is a core disk forensics tool** from The Sleuth Kit that parses partition tables and reports partition boundaries in sectors. It works on raw disk images without needing to mount them.
```
