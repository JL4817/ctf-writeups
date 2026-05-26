# substitution0

## Challenge Overview

The challenge provides an **encrypted file** with a strange encoding.
The goal is to decrypt it and find the flag.

---

## Exploitation

### Step 1 — Identify the Cipher

The ciphertext had a suspicious structure — it looked like a letter shift of some kind.

**Caesar cipher** was the first attempt, trying all 26 rotations — none produced readable output.

**Vigenère cipher** was also ruled out.

This pointed to a **Monoalphabetic Substitution Cipher**, where each letter maps to a unique different letter consistently throughout the text — but not with a fixed shift.

### Step 2 — Known Plaintext Crib

The flag format `picoCTF{...}` gave an immediate foothold. Matching it against the ciphertext:

```
Ciphertext: xqcpCBM{5UE5717U710Z_3S0WU710Z_59533D2F}
Plaintext:  picoCTF{...}
```

This immediately revealed 7 character mappings:

```
x → p
q → i
c → c
p → o
C → C
B → T
M → F
```

### Step 3 — Build the Full Mapping

Using a **Monoalphabetic Substitution Cipher solver** on the terminal, we fed in the ciphertext and used the known flag format as a crib. Each substitution revealed more of the plaintext, which in turn revealed more mappings, until the full alphabet was reconstructed.

🏁 Flag retrieved.

---

## Key Takeaway

> **Monoalphabetic substitution is vulnerable to frequency analysis and known-plaintext attacks.** Knowing the flag format (e.g. `picoCTF{...}`) gives an immediate crib that unravels the entire substitution alphabet.
```
