# Mini RSA

## Challenge Overview

The challenge provides an **RSA-encrypted message** with a dangerously small public exponent.

```
N: 16157656843214630540782260519598878842336783177348929017407633211352136367960754...
e: 3
c: 57097201750263178419445051663321827794197600311154322155634259018756200527916081...
```

---

## Vulnerability

Standard RSA decryption requires the private key `d`:

```
M = c^d mod N
```

But here `e = 3`, which means encryption was:

```
c = M^e mod N
c = M^3 mod N
```

If the message `M` is small enough, then **M³ < N**, meaning the `mod N` never activates.
This simplifies the problem dramatically:

```
c = M^3
M = ∛c
```

No private key needed — we just take the **cube root of the ciphertext**.

---

## Exploitation

### Step 1 — Why Not Just Use Python's Built-in?

The naive approach fails for huge numbers because Python's float math loses precision:

```python
m = int(c ** (1/3))  # ❌ inaccurate for large integers
```

### Step 2 — Implement an Exact Integer Cube Root

We use a **binary search** to find the exact integer cube root:

```
mid^3 == c  →  found it
mid^3 < c   →  mid is too small, search higher
mid^3 > c   →  mid is too large, search lower
```

```python
def cube_root(n):
    low = 0
    high = n

    while low <= high:
        mid = (low + high) // 2
        cube = mid ** 3

        if cube == n:
            return mid
        elif cube < n:
            low = mid + 1
        else:
            high = mid - 1

    return high
```

### Step 3 — Recover the Message

```python
m = cube_root(c)

# Verify
print(m ** 3 == c)  # True ✅

# Convert integer to bytes and decode
message = m.to_bytes((m.bit_length() + 7) // 8, "big")
print(message.decode())
```

🏁 Flag retrieved.

---

## Key Takeaway

> **Never use a small public exponent like `e = 3` without proper padding (e.g. OAEP).** If the message is small relative to `N`, the modular reduction never applies and the ciphertext is just `M^e` in plain integers — trivially reversible with a cube root.
```
