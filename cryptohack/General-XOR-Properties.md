# XOR Properties

## Challenge Overview

The challenge provides a chain of XOR'd values built from three keys and a flag. The goal is to use the algebraic properties of XOR to isolate and recover each key, then decrypt the flag.

```
KEY1                       = a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313
KEY2 ^ KEY1                = 37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e
KEY2 ^ KEY3                = c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1
FLAG ^ KEY1 ^ KEY3 ^ KEY2  = 04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf
```

---

## XOR Properties

| Property | Rule |
|----------|------|
| Commutative | `A ⊕ B = B ⊕ A` |
| Associative | `A ⊕ (B ⊕ C) = (A ⊕ B) ⊕ C` |
| Identity | `A ⊕ 0 = A` |
| Self-Inverse | `A ⊕ A = 0` |

These four properties together mean we can **reorder and cancel terms freely** in a chain of XOR operations — exactly like algebra, but with XOR instead of addition.

---

## Exploitation

### Step 1 — Recover KEY2

We're given `KEY2 ^ KEY1` and `KEY1`. XOR-ing both together cancels `KEY1`:

```
(KEY2 ^ KEY1) ^ KEY1
= KEY2 ^ (KEY1 ^ KEY1)
= KEY2 ^ 0
= KEY2
```

### Step 2 — Recover KEY3

Now that we have `KEY2`, we XOR it with `KEY2 ^ KEY3` to cancel `KEY2`:

```
(KEY2 ^ KEY3) ^ KEY2 = KEY3
```

### Step 3 — Recover the Flag

With `KEY1`, `KEY2`, and `KEY3` known, we XOR them all against the final encrypted line to cancel out every key:

```
(FLAG ^ KEY1 ^ KEY3 ^ KEY2) ^ KEY1 ^ KEY2 ^ KEY3 = FLAG
```

### Step 4 — Implement in Python

```python
from pwn import xor

key1 = bytes.fromhex("a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313")
key2_xor_key1 = bytes.fromhex("37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e")
key2_xor_key3 = bytes.fromhex("c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1")
flag_xor_all = bytes.fromhex("04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf")

key2 = xor(key2_xor_key1, key1)
key3 = xor(key2_xor_key3, key2)
flag = xor(flag_xor_all, key1, key2, key3)

print(flag.decode())
```

🏁 Flag retrieved.

---

## Key Takeaway

> **XOR's algebraic properties make it possible to "undo" a chain of XOR operations** without ever needing the original values directly exposed — as long as enough overlapping combinations are known. This is the foundation for attacking many real-world stream ciphers and one-time pad misuse.
```
