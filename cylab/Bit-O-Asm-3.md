Here you go:

```markdown
# Bit-O-Asm-3

## Challenge Overview

The challenge provides an **x86-64 assembly dump** and asks for the final value stored in the `eax` register, converted to decimal.

---

## Exploitation

### Step 1 — Strip the Boilerplate

The following instructions are standard function prologue and unused argument saves — they don't affect the result:

```asm
endbr64
push   rbp
mov    rbp, rsp
mov    DWORD PTR [rbp-0x14], edi   ; arg1 saved but unused
mov    QWORD PTR [rbp-0x20], rsi   ; arg2 saved but unused
```

### Step 2 — Trace the Important Instructions

```asm
mov  DWORD PTR [rbp-0xc], 0x9fe1a  ; a = 0x9fe1a
mov  DWORD PTR [rbp-0x8], 0x4      ; b = 0x4
mov  eax, DWORD PTR [rbp-0xc]      ; eax = a
imul eax, DWORD PTR [rbp-0x8]      ; eax = eax * b
add  eax, 0x1f5                    ; eax = eax + 0x1f5
mov  DWORD PTR [rbp-0x4], eax      ; store result
mov  eax, DWORD PTR [rbp-0x4]      ; return result in eax
```

This is equivalent to:

```c
int a      = 0x9fe1a;
int b      = 0x4;
int result = a * b + 0x1f5;
return result;
```

### Step 3 — Convert and Calculate

```
0x9fe1a = 654874
0x1f5   = 501
0x4     = 4

654874 × 4 = 2619496
2619496 + 501 = 2619997
```

✅ `eax = 2619997`

🏁 Flag: `picoCTF{2619997}`

---

## Key Takeaway

> **Reading assembly is just tracking values through registers and memory.** Ignore the prologue, identify which variables are actually used, then follow the arithmetic step by step. `imul` is signed multiply, `add` is plain addition — translating to C-like pseudocode makes the logic immediately clear.
```
