# PW Crack 4

## Challenge Overview

Same setup as the previous challenge — but this time the password list contains **100 candidates** instead of 7, making manual attempts impractical.

---

## Exploitation

The approach is identical to before, but automation is now essential.

We appended a brute-force loop to the script that hashes every candidate and compares it against the correct hash:

```python
for user_pw in pos_pw_list:
    user_pw_hash = hash_pw(user_pw)

    if user_pw_hash == correct_pw_hash:
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), user_pw)
        print(decryption)
        break
```

Running the script iterated through all 100 candidates, found the matching hash, XOR-decrypted the flag, and printed it.

🏁 Flag retrieved.

---

## Key Takeaway

> **A larger password list offers no real security if it is hardcoded alongside the encrypted data.** Whether 7 or 100 entries, a simple loop cracks it instantly — true security requires passwords that are not distributed with the challenge.
```
