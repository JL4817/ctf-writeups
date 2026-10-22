# What Lies Within

## Challenge Overview

The challenge provides a **PNG image** (`buildings.png`) with a flag hidden inside it using **LSB (Least Significant Bit) steganography**.

---

## Exploitation

### Step 1 — Identify the Steganography Technique

The hint suggests something is hidden inside the image. LSB steganography works by replacing the least significant bit of each colour channel's byte with a bit of the secret message — a change so small it is invisible to the human eye.

### Step 2 — Extract with zsteg

`zsteg` is a tool that detects and extracts data hidden in PNG files across multiple bit planes and colour channels:

```bash
zsteg -a buildings.png
```

```
b1,rgb,lsb,xy    .. text: "academy{h1d1ng_1n_th3_b1t5}"
b2,b,lsb,xy      .. text: "XuH}p#8Iy="
b3,abgr,msb,xy   .. text: "t@Wp-_tH_v"
```

The first result — `b1,rgb,lsb,xy` — is the standard LSB channel and contains the flag.

🏁 Flag: `academy{h1d1ng_1n_th3_b1t5}`

---

## Key Takeaway

> **LSB steganography hides data in the least significant bits of pixel colour values**, making the change visually imperceptible. `zsteg` automates the search across all common bit planes and channel combinations, making it the go-to tool for PNG steganography challenges.
```
