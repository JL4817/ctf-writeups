# m00nwalk

## Challenge Overview

The challenge provides a **`message.wav`** audio file. The goal is to extract a hidden flag from it.

---

## Exploitation

### Step 1 — Initial Inspection

Opening the file in a music app revealed a UUID-like string in the metadata:

```
d27864c3-52a5-438b-931a-7f20a6e479ed
```

This didn't seem relevant. We also checked for plaintext strings and metadata:

```bash
strings message.wav | grep -i flag
exiftool message.wav | grep -i flag
```

Both returned nothing useful.

### Step 2 — Identify the Encoding

Steganography tools like `steghide` hide data silently — but **listening to the audio**, it sounded like distorted screaming/noise rather than a normal audio file.

This is a recognizable characteristic of **SSTV (Slow Scan Television)** — a method of transmitting images over audio signals. The "noise" is actually an encoded image.

### Step 3 — Decode with SSTV

We decoded the audio using an SSTV decoder, which converted the signal into an image:

```bash
# Example using rx_sstv or a tool like QSSTV / Black Cat SSTV
rx_sstv message.wav
```

The output image contained the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **Steganography is not always visual.** SSTV encodes images inside audio signals — if a `.wav` file sounds like harsh static or mechanical noise, consider running it through an SSTV decoder before reaching for `steghide`.
```
