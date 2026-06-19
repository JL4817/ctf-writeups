# Encoding Challenge — 100 Levels Automation

## Challenge Overview

The challenge provides a **server-side Python script** (`13377.py`) that generates 100 sequential encoding levels. Each level sends a string encoded in one of five formats — the goal is to decode it correctly and send it back, automating the process to pass all 100 levels and retrieve the flag.

```python
ENCODINGS = ["base64", "hex", "rot13", "bigint", "utf-8"]
```

---

## Exploitation

### Step 1 — Understand the Server Logic

Each level:
1. Picks 3 random words and joins them with underscores
2. Randomly encodes them in one of the 5 formats
3. Sends `{"type": encoding, "encoded": value}`
4. Expects `{"decoded": original_string}` back
5. On correct answer → advances to the next level
6. After level 100 → returns the flag

### Step 2 — Write a Decoder for Each Encoding

```python
def decode_value(encoding_type, encoded_value):
    if encoding_type == "base64":
        return base64.b64decode(encoded_value).decode()
    elif encoding_type == "hex":
        return bytes.fromhex(encoded_value).decode()
    elif encoding_type == "rot13":
        return codecs.decode(encoded_value, 'rot_13')
    elif encoding_type == "bigint":
        hex_value = encoded_value.replace('0x', '')
        return long_to_bytes(int(hex_value, 16)).decode()
    elif encoding_type == "utf-8":
        return ''.join(chr(b) for b in encoded_value)
    else:
        raise ValueError(f"Unknown encoding: {encoding_type}")
```

### Step 3 — Automate the Full Exchange

Using `pwntools` to connect and loop through all 100 levels automatically:

```python
from pwn import *
import json, base64, codecs
from Crypto.Util.number import long_to_bytes

r = remote('socket.cryptohack.org', 13377, level='debug')

def json_recv():
    return json.loads(r.recvline().decode())

def json_send(hsh):
    r.sendline(json.dumps(hsh).encode())

received = json_recv()

for level in range(100):
    encoding_type = received["type"]
    encoded_value = received["encoded"]
    decoded = decode_value(encoding_type, encoded_value)

    print(f"Level {level}: {encoding_type} -> {decoded[:50]}...")

    json_send({"decoded": decoded})
    received = json_recv()

    if "flag" in received:
        print(f"\n🎉 FLAG: {received['flag']}")
        break

r.close()
```

### Step 4 — Run the Script

Running the script connects to the server, automatically decodes and resubmits each of the 100 levels, and prints the flag once all levels pass.

🏁 Flag retrieved.

---

## Key Takeaway

> **Automation is essential for multi-step challenges.** Writing a small decoder function per encoding type and looping through the exchange with `pwntools` turns a tedious 100-step manual task into a script that runs in seconds.
```
