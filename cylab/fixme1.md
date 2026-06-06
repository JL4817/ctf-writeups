# fixme1

## Challenge Overview

The challenge provides a **Python script** with a subtle syntax error preventing it from running.
Fix the error and the script prints the flag.

---

## Vulnerability

The problematic code:

```python
flag = str_xor(flag_enc, 'enkidu')
  print('That is correct! Here\'s your flag: ' + flag)
```

The `print` statement has an **unexpected indentation** — it is indented but not inside any block (no `if`, `for`, `def`, etc. above it). In Python, indentation is **syntax**, not just style.

---

## Python Indentation vs Other Languages

Other languages use braces `{}` to define blocks:

```java
if (x > 0) {
    System.out.println("positive");
    System.out.println("still inside if");
}
System.out.println("outside if");
```

Python uses **indentation** instead:

```python
if x > 0:
    print("positive")        # inside if
    print("still inside if") # inside if
print("outside if")          # outside if
```

An unexpected indent where no block exists is a hard syntax error — Python refuses to run the file at all.

---

## Fix

Remove the extra indentation from the `print` statement:

```python
flag = str_xor(flag_enc, 'enkidu')
print('That is correct! Here\'s your flag: ' + flag)
```

Running the corrected script decrypts the flag using XOR with the key `enkidu` and prints it.

🏁 Flag retrieved.

---

## Key Takeaway

> **In Python, indentation is syntax.** An unexpected indent crashes the entire program before it even runs. Always ensure `print` and other statements are only indented when they genuinely belong inside a block.
```
