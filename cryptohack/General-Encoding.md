# General-Encoding (except Encoding Challenge) 

## Challenge Overview

Four introductory encoding challenges, each requiring a different decoding technique to recover the flag.

---

## Challenge 1 — ASCII

Convert an array of integers to their corresponding ASCII characters:

```
[99, 114, 121, 112, 116, 111, 123, 65, 83, 67, 73, 73, 95, 112, 114, 49, 110, 116, 52, 98, 108, 51, 125]
```

```python
arr = [99, 114, 121, 112, 116, 111, 123, 65, 83, 67, 73, 73, 95, 112, 114, 49, 110, 116, 52, 98, 108, 51, 125]
result = ""
for x in arr:
    result += chr(x)
print(result)
```

Each integer maps directly to an ASCII character via `chr()`.

🏁 Flag retrieved.

---

## Challenge 2 — Hex

Decode a hex string back into readable text:

```
63727970746f7b596f755f77696c6c5f62655f776f726b696e675f776974685f6865785f737472696e67735f615f6c6f747d
```

```python
hex_string = "63727970746f7b596f755f77696c6c5f62655f776f726b696e675f776974685f6865785f737472696e67735f615f6c6f747d"
result = bytes.fromhex(hex_string).decode()
print(result)
```

`bytes.fromhex()` converts the hex string to raw bytes, then `.decode()` interprets them as UTF-8 text.

🏁 Flag retrieved.

---

## Challenge 3 — Base64

Convert a hex string to bytes, then encode as Base64:

```
72bca9b68fc16ac7beeb8f849dca1d8a783e8acf9679bf9269f7bf
```

```python
import base64

hex_string = "72bca9b68fc16ac7beeb8f849dca1d8a783e8acf9679bf9269f7bf"

# Step 1: hex → bytes
data = bytes.fromhex(hex_string)

# Step 2: bytes → Base64
result = base64.b64encode(data).decode()
print(result)
```

🏁 Flag retrieved.

---

## Challenge 4 — Bytes and Big Integers

A large integer encodes a message. Convert it back to bytes then to text:

```python
from Crypto.Util.number import long_to_bytes

num = 11515195063862318899931685488813747395775516287289682636499965282714637259206269
message = long_to_bytes(num)
print(message.decode())
```

`long_to_bytes()` from `pycryptodome` interprets the integer as a big-endian byte sequence and converts it back to readable text.

🏁 Flag retrieved.

---

## Key Takeaway

> **Encoding is not encryption.** ASCII, hex, Base64, and big-integer representations are all reversible without any key — they are just different ways of representing the same data. Recognising which encoding is in use is a fundamental CTF skill.
```
