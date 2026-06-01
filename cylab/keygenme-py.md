# keygenme-py

## Challenge Overview

The challenge provides a **game binary with a license key check**. The key has two parts — a static prefix and a dynamic section derived from a username hash.

```python
key_part_static1_trial  = "picoCTF{1n_7h3_kk3y_of_"
key_part_dynamic1_trial = "xxxxxxxx"
key_part_static2_trial  = "}"
```

The goal is to reverse the `check_key` function to reconstruct `key_part_dynamic1_trial`.

---

## Exploitation

### Step 1 — Analyse check_key()

The function validates the dynamic part by picking specific characters from the SHA-256 hash of the username, **in a non-sequential order**:

```python
if key[i] != hashlib.sha256(username_trial).hexdigest()[4]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[5]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[3]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[6]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[2]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[7]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[1]: return False
if key[i] != hashlib.sha256(username_trial).hexdigest()[8]: return False
```

Index order: `[4], [5], [3], [6], [2], [7], [1], [8]`

### Step 2 — Spot the Bug

The source defines:

```python
username_trial = "BENNETT"   # str
bUsername_trial = b"BENNETT" # bytes
```

But `enter_license()` passes `bUsername_trial` (bytes) into `check_key()` — so the hash must be computed on the **bytes** version:

```python
hashlib.sha256(b"BENNETT").hexdigest()
```

### Step 3 — Compute the Hash

```python
import hashlib
print(hashlib.sha256(b"BENNETT").hexdigest())
# ba6c084a4d888e1f7c3b0fc71d61c4625708bd915b5e0e60eb73e1667251b567
```

### Step 4 — Extract the Dynamic Key

Picking characters at indexes `[4], [5], [3], [6], [2], [7], [1], [8]`:

```
Index:  0  1  2  3  4  5  6  7  8  9 ...
Hash:   b  a  6  c  0  8  4  a  4  d ...

[4] = 0
[5] = 8
[3] = c
[6] = 4
[2] = 6
[7] = a
[1] = a
[8] = 4
```

Dynamic part: `08c46aa4`

### Step 5 — Assemble the Flag

```
picoCTF{1n_7h3_kk3y_of_08c46aa4}
```

🏁 Flag retrieved.

---

## Key Takeaway

> **Security through obscurity is not security.** Scrambling the index order of a hash does not hide the key — anyone who reads the source code can trivially reconstruct it in seconds.
```
