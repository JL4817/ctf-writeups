# CryptoHack Introduction

## Challenge Overview

Three introductory challenges covering basic CTF skills: finding flags, running Python scripts, and communicating over a network socket.

---

## Challenge 1 — Finding Flags

The flag was already present on the page. Simply copy and submit it.

🏁 Flag retrieved.

---

## Challenge 2 — Great Snakes

A Python file is provided. Running it directly prints the flag:

```bash
python3 great_snakes.py
```

🏁 Flag retrieved.

---

## Challenge 3 — Network Attacks

The challenge requires connecting to a socket server and sending a JSON object to retrieve the flag.

**Goal:** Connect to `socket.cryptohack.org` on port `11112` and send:
```json
{ "buy": "flag" }
```

### Attempt 1 — pwntools

The provided template uses `pwntools`:

```python
from pwn import *
import json

HOST = "socket.cryptohack.org"
PORT = 11112

r = remote(HOST, PORT)
request = {"buy": "flag"}
r.sendline(json.dumps(request).encode())
response = json.loads(r.readline().decode())
print(response)
```

### Attempt 2 — Raw Socket (no dependencies)

Instead of installing pwntools, we wrote a minimal solution using Python's built-in `socket` library:

```python
import socket
import json

HOST = "socket.cryptohack.org"
PORT = 11112

s = socket.create_connection((HOST, PORT))

# Read banner
print(s.recv(4096).decode())

# Send the request
request = {"buy": "flag"}
s.sendall(json.dumps(request).encode() + b"\n")

# Receive the flag
response = s.recv(4096).decode()
print(response)
```

Running this connects to the server, reads the banner, sends the correct JSON payload, and prints the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **CTF network challenges almost always follow the same pattern:** read the banner, send a crafted payload, receive the response. Python's built-in `socket` library is often enough — no third-party tools required.
```
