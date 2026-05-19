# RSA

## Challenge Overview

The challenge provides an **RSA-encrypted message** and asks us to decrypt it.

**RSA** is a public-key cryptosystem where encryption is done with a public key `(n, e)` and decryption requires the private key `d`.

```
c: 15341890103764929939105506004034128738090325640037083301857608662849501626260517
n: 948406957756830799684818171639547165784816468744946013083947881743680617123566349
e: 65537
```

---

## Exploitation

### Step 1 — Factor n into p and q

RSA security relies on `n = p * q` being hard to factor. Since this `n` is small enough, we can factor it directly:

```
p = 1891771437429478964908181306574287207137
q = 501332739776173570344039681219489434626477
```

```python
assert p * q == n  # ✅ confirmed
```

### Step 2 — Compute phi

With `p` and `q` known, we can compute Euler's totient:

```
phi = (p - 1) * (q - 1)
```

### Step 3 — Compute the Private Key d

The private exponent `d` is the modular inverse of `e` mod `phi`:

```
d = e⁻¹ mod phi
```

```python
d = pow(e, -1, phi)
```

### Step 4 — Decrypt the Ciphertext

With `d` known, we decrypt using:

```
m = c^d mod n
```

```python
m = pow(c, d, n)
```

### Step 5 — Convert m to Bytes

```python
msg = m.to_bytes((m.bit_length() + 7) // 8, "big")
```

### Step 6 — Reverse the Result

The message was encrypted backwards, so we reverse the bytes to get the flag:

```python
print(msg[::-1].decode())
```

---

## Full Script

```python
c = 15341890103764929939105506004034128738090325640037083301857608662849501626260517
n = 948406957756830799684818171639547165784816468744946013083947881743680617123566349
e = 65537

p = 1891771437429478964908181306574287207137
q = 501332739776173570344039681219489434626477

assert p * q == n

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)

msg = m.to_bytes((m.bit_length() + 7) // 8, "big")
print(msg[::-1].decode())
```

🏁 Flag retrieved.
```
