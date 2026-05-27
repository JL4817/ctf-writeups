# PW Crack 3

## Challenge Overview

The challenge provides three files:

```
level3.flag.txt.enc   — encrypted flag
level3.hash.bin       — correct password hash
level3.py             — encryption/decryption logic
```

---

## Exploitation

### Step 1 — Analyse the Source Code

Reading `level3.py`, the logic was sound — no vulnerabilities in the encryption itself. However, the list of **possible passwords was hardcoded** at the bottom of the script, leaving only 7 candidates to try.

### Step 2 — Brute Force the Password List

Rather than trying each password manually, we appended a brute-force loop to the script that hashes every candidate and compares it against the correct hash:

```python
for pw in pos_pw_list:
    user_pw_hash = hash_pw(pw)
    if user_pw_hash == correct_pw_hash:
        print(f"Correct Password Found: {pw}")
        decryption = str_xor(flag_enc.decode(), pw)
        print("Flag:", decryption)
        break
```

### Step 3 — Run the Script

Running the modified script iterated through all 7 candidates, found the matching hash, XOR-decrypted the flag, and printed it.

🏁 Flag retrieved.

---

## Key Takeaway

> **Never hardcode a finite password list alongside encrypted data.** Even with hashing, a small candidate list is trivially brute-forced — an attacker only needs to hash each candidate and compare.
```
