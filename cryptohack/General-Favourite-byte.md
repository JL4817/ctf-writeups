# Favourite Byte

## Challenge Overview

The challenge provides a **hex-encoded string** that has been XOR'd with a single secret byte. The goal is to recover that byte and decode the message.

```
73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d
```

---

## Concept — Single-Byte XOR

A single byte can hold **256 possible values** (0–255). Since the key is unknown but bounded, we can simply **brute-force every possible key** and check which one produces readable text — this is a classic single-byte XOR brute force.

---

## Exploitation

### Step 1 — Decode from Hex

```python
encoded = bytes.fromhex("73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d")
```

### Step 2 — Brute Force All 256 Keys

For each possible byte value, we XOR it against every byte of the message and print the result:

```python
for key in range(256):
    decoded = ""
    for b in encoded:
        decoded += chr(b ^ key)
    print(key, decoded)
```

### Step 3 — Identify the Correct Key

Scanning through the 256 outputs, one key produces a readable plaintext string containing the flag (the rest produce garbage/unprintable characters).

🏁 Flag retrieved.

---

## Key Takeaway

> **Single-byte XOR has only 256 possible keys**, making it trivially brute-forceable by a computer. This is far weaker than a repeating-key or one-time-pad XOR — any single-byte XOR cipher can be broken in milliseconds regardless of message length.
```
