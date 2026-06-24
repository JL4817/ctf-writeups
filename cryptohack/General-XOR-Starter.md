# General: XOR-Starter

## Challenge Overview

The challenge introduces the **XOR (exclusive OR) bitwise operator** and asks us to apply it to a string.

**XOR truth table:**

| A | B | Output |
|---|---|--------|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

For longer binary values, XOR is applied bit by bit:
```
0110 ^ 1010 = 1100
```

**Goal:** XOR each character of the string `"label"` with the integer `13`, then convert the result back to a string to get the flag.

---

## Exploitation

### Step 1 — Convert Characters to Integers

Each character is converted to its Unicode code point using `ord()`.

### Step 2 — XOR Each Character

We XOR every character's code point with `13` using Python's `^` operator:

```python
word = "label"
answer = ""

for c in word:
    answer += chr(ord(c) ^ 13)

print(answer)
```

### Step 3 — Build the Flag

The resulting string is wrapped in the flag format:

```
crypto{<answer>}
```

🏁 Flag retrieved.

---

## Key Takeaway

> **XOR is reversible by design** — XOR-ing the same value twice returns the original input: `(A ^ B) ^ B = A`. This property makes it the foundation of many encryption schemes (like one-time pads and stream ciphers), but using a single repeated key (like `13`) provides no real security.
```
