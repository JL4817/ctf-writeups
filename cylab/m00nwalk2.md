# m00nwalk2

## Challenge Overview

The challenge provides **four audio files**: `clue1.wav`, `clue2.wav`, `clue3.wav`, and `message.wav`.
The goal is to extract clues from the first three and use them to decrypt the final message.

---

## Exploitation

### Step 1 — Initial Inspection

Standard checks on the audio files returned nothing:

```bash
strings clue1.wav | grep -i flag
exiftool clue1.wav | grep -i flag
```

The audio sounded like harsh static noise — a recognizable characteristic of **SSTV (Slow Scan Television)**, a method of encoding images inside audio signals.

### Step 2 — Decode the SSTV Audio Files

We cloned and set up the SSTV decoder:

```bash
git clone https://github.com/colaclanth/sstv.git
cd sstv
python3 -m venv venv
source venv/bin/activate
```

Then decoded all three clue files:

```bash
sstv -d clue1.wav -o result.png
# Detected SSTV mode: Martin 1

sstv -d clue2.wav -o result1.png
# Detected SSTV mode: Scottie 2

sstv -d clue3.wav -o result2.png
# Detected SSTV mode: Martin 2
```

Each produced a different image.

### Step 3 — Analyse the Images

Examining the three decoded images:

- One contained the phrase **"Alan Eliasen the Future Boy"**
- One contained a **password**
- One contained additional visual clues

Searching "Alan Eliasen the Future Boy" on Google leads to **futureboy.us** — a site hosting steganography tools, including a WAV steganography decoder.

### Step 4 — Decrypt message.wav

Using the steganography tool at **futureboy.us** with:
- **Input:** `message.wav`
- **Password:** (found in one of the decoded images)

The tool decoded the hidden message from the audio and returned the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **Steganography can be layered.** Here, SSTV was used to hide images inside audio, and those images contained the password for a second layer of audio steganography. Always check if extracted clues point to further tools or decryption steps.
```
