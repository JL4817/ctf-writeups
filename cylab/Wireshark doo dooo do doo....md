# Wireshark doo dooo do doo...

## Challenge Overview

The challenge provides a **packet capture file** (`shark1.pcapng`). The goal is to extract and decode a hidden flag from the network traffic.

---

## Exploitation

### Step 1 — Extract the TCP Stream

Using `tshark` to follow TCP stream 5:

```bash
tshark -r shark1.pcapng -q -z follow,tcp,ascii,5
```

The HTTP response body contains:

```
Gur synt vf npnqrzl{c33xno00_1_f33_h_qrnqorrs}
```

### Step 2 — Identify the Encoding

The text looks like readable English but with shifted letters — a classic **ROT13** Caesar cipher (shift of 13).

### Step 3 — Decode ROT13

```bash
echo "Gur synt vf npnqrzl{c33xno00_1_f33_h_qrnqorrs}" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

```
The flag is academy{p33kab00_1_s33_u_deadbeef}
```

🏁 Flag: `academy{p33kab00_1_s33_u_deadbeef}`

---

## Key Takeaway

> **ROT13 is a trivial Caesar cipher that shifts each letter by 13 positions** — applying it twice returns the original text. It provides zero security and is trivially reversible with any online tool or `tr` in the terminal. Always check HTTP response bodies in packet captures for encoded or plaintext flags.
```
