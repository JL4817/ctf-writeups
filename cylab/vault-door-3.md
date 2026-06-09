# vault-door-3

## Challenge Overview

The challenge provides **Java source code** that scrambles a password into a known output string.
The goal is to reverse the scrambling operations to recover the original password (the flag).

```
Scrambled output: jU5t_a_sna_3lpm16g041_u_4_m2r547
```

---

## Vulnerability

The source code applies four scrambling passes to rearrange characters into a 32-character buffer:

```java
char[] buffer = new char[32];
int i;

// Pass 1: copy first 8 chars directly
for (i=0; i<8; i++) {
    buffer[i] = password.charAt(i);
}
// Pass 2: reverse mapping for indexes 8–15
for (; i<16; i++) {
    buffer[i] = password.charAt(23-i);
}
// Pass 3: even indexes 16–31, reverse mapping
for (; i<32; i+=2) {
    buffer[i] = password.charAt(46-i);
}
// Pass 4: odd indexes 31–17, direct mapping
for (i=31; i>=17; i-=2) {
    buffer[i] = password.charAt(i);
}
```

Since all four operations are **deterministic index mappings** with no data loss, they can be fully reversed.

---

## Exploitation

### Step 1 — Reverse Each Pass

For each pass we swap the direction of the mapping — instead of `buffer[i] = password[f(i)]`, we do `password[f(i)] = buffer[i]`:

```java
public class VaultDoor3Solver {
    public static void main(String[] args) {
        String s = "jU5t_a_sna_3lpm16g041_u_4_m2r547";
        char[] password = new char[32];

        // Reverse Pass 1: buffer[i] = password[i]
        for (int i = 0; i < 8; i++) {
            password[i] = s.charAt(i);
        }

        // Reverse Pass 2: buffer[i] = password[23-i]  →  password[23-i] = buffer[i]
        for (int i = 8; i < 16; i++) {
            password[23 - i] = s.charAt(i);
        }

        // Reverse Pass 3: buffer[i] = password[46-i]  →  password[46-i] = buffer[i]
        for (int i = 16; i < 32; i += 2) {
            password[46 - i] = s.charAt(i);
        }

        // Reverse Pass 4: buffer[i] = password[i]
        for (int i = 31; i >= 17; i -= 2) {
            password[i] = s.charAt(i);
        }

        System.out.println(new String(password));
    }
}
```

### Step 2 — Run the Solver

Compiling and running the Java solver reconstructs the original password and prints the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **Scrambling is not encryption.** Rearranging characters with a fixed index mapping is fully reversible without any key — anyone with the source code can reconstruct the original input in seconds. Real security requires cryptographic primitives, not permutations.
```
