# vault-door-4

## Challenge Overview

The challenge provides **Java source code** with a hardcoded byte array that the password is compared against.
The goal is to decode the byte array to recover the flag.

---

## Vulnerability

The `checkPassword` function simply compares the input byte-by-byte against a hardcoded array:

```java
public boolean checkPassword(String password) {
    byte[] passBytes = password.getBytes();
    byte[] myBytes = {
        106 , 85  , 53  , 116 , 95  , 52  , 95  , 98  ,  // decimal
        0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,  // hex
        0142, 0131, 0164, 063 , 0163, 0137, 061 , 063 ,   // octal
        'd' , 'f' , '6' , '1' , '8' , 'a' , '2' , '3' ,  // char literals
    };
    for (int i=0; i<32; i++) {
        if (passBytes[i] != myBytes[i]) {
            return false;
        }
    }
    return true;
}
```

The bytes are written in **four different number bases** which are decimal, hexadecimal, octal, and character literals, but they all map directly to ASCII/Unicode characters. There is no encryption; the password is sitting right there in the source code.

---

## Exploitation

### Step 1 — Decode the Byte Array

Each group uses a different radix but all decode to printable ASCII:

| Group | Format | Example |
|-------|--------|---------|
| 1–8   | Decimal | `106` → `j` |
| 9–16  | Hex | `0x55` → `U` |
| 17–24 | Octal | `0142` → `b` |
| 25–32 | Char literals | `'d'` → `d` |

### Step 2 — Print the Flag Directly

We cast each byte to a `char` and print it:

```java
byte[] myBytes = {
    106 , 85  , 53  , 116 , 95  , 52  , 95  , 98  ,
    0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
    0142, 0131, 0164, 063 , 0163, 0137, 061 , 063 ,
    'd' , 'f' , '6' , '1' , '8' , 'a' , '2' , '3' ,
};

for (byte b : myBytes) {
    System.out.print((char) b);
}
System.out.println();
```

Running this prints the flag directly.

🏁 Flag retrieved.

---

## Key Takeaway

> **Obfuscating numbers across different bases (decimal, hex, octal) provides zero security.** All three are just different ways of writing the same integer values — a compiler treats them identically. Never hardcode passwords or flags in source code regardless of how they are formatted.
```
