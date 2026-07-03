# You Either Know, XOR You Don't

## Challenge Overview

The challenge provides a **hex-encoded ciphertext** encrypted with a repeating XOR key. The goal is to recover the key and decrypt the flag.

```
0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104
```

---

## Concept — Known Plaintext Attack

Since all flags begin with `crypto{`, we have a **known plaintext** for the first 7 bytes. Using XOR's self-inverse property:

```
cipher ^ plaintext = key
```

We can XOR the known plaintext bytes against the corresponding ciphertext bytes to recover part of the key.

---

## Exploitation

### Step 1 — Recover the Key Using Known Plaintext

```python
cipher = bytes.fromhex(
    "0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104"
)

known = b"crypto{"

key = bytes(c ^ p for c, p in zip(cipher, known))
print(key)
```

This recovers the first 7 bytes of the key:

```
myXORke
```

Which we can infer is the full repeating key: **`myXORkey`**

### Step 2 — Decrypt the Full Ciphertext

With the key known, we XOR every byte of the ciphertext against the repeating key:

```python
key = b"myXORkey"
answer = ""

for i in range(len(cipher)):
    cipher_byte = cipher[i]
    key_byte = key[i % len(key)]   # repeat key cyclically
    decrypted_byte = cipher_byte ^ key_byte
    answer += chr(decrypted_byte)

print(answer)
```

🏁 Flag retrieved.

---

## Key Takeaway

> **Repeating-key XOR is vulnerable to known-plaintext attacks.** Any predictable prefix — like a flag format — immediately leaks key bytes. Once even a few key bytes are recovered, patterns in the key (like a real word) allow the full key to be guessed. Always use a key at least as long as the message (a true one-time pad) to prevent this.
```
