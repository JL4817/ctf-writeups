# PW Crack 5

## Challenge Overview

Same setup as before — but this time the possible passwords are in a **`dictionary.txt`** file containing thousands of entries, making an in-memory array impractical.

---

## Exploitation

### Step 1 — The Problem with Previous Approaches

In the earlier challenges, passwords were hardcoded in a list (7 and then 100 entries). With thousands of passwords, we need to **read the file line by line** instead of loading everything into memory at once.

### Step 2 — Dictionary Attack

We wrote a solver that opens `dictionary.txt` and tries each password:

```python
with open("dictionary.txt", "r", encoding="utf-8", errors="ignore") as f:
    for user_pw in f:
        user_pw = user_pw.strip()

        if hash_pw(user_pw) == correct_pw_hash:
            print("Password found:", user_pw)
            decryption = str_xor(flag_enc.decode(), user_pw)
            print("Flag:")
            print(decryption)
            break
    else:
        print("Password not found in dictionary.")
```

**What each part does:**

- `open(..., encoding="utf-8", errors="ignore")` — reads the file in UTF-8, skipping any invalid characters
- `strip()` — removes the trailing newline `\n` from each password
- `hash_pw(user_pw) == correct_pw_hash` — hashes each candidate with MD5 and compares against the target hash
- `break` — stops as soon as the correct password is found
- `else` — only runs if the entire dictionary was exhausted with no match

Running the script found the matching password, XOR-decrypted the flag, and printed it.

🏁 Flag retrieved.

---

## Key Takeaway

> **Dictionary attacks are highly effective against weak or common passwords.** Even thousands of candidates can be cracked in seconds. Passwords should be long, random, and never appear in any wordlist.
```
