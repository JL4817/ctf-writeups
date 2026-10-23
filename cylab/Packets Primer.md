# Packets Primer

## Challenge Overview

The challenge provides a **packet capture file** (`.pcap`). The goal is to analyse the network traffic and extract the hidden flag.

---

## Exploitation

### Step 1 — Open in Wireshark

Open the capture file in Wireshark and examine the packet list. The capture contains a standard TCP handshake followed by a data transfer:

| Packet | TCP Info | Meaning |
|--------|----------|---------|
| 1 | SYN, Len=0 | Client requests connection |
| 2 | SYN, ACK, Len=0 | Server accepts |
| 3 | ACK, Len=0 | Client confirms |
| 4 | **PSH, ACK, Len=60** | Client sends 60 bytes of data ← |
| 5 | ACK, Len=0 | Server acknowledges |

Packets 1–3 are the TCP handshake (no data). Packet 5 is just an acknowledgement. **Packet 4** is the only one carrying actual payload data.

### Step 2 — Follow the TCP Stream

Right-click any packet → **Follow → TCP Stream**. This reassembles the raw data from packet 4 and displays the flag in plaintext.

🏁 Flag retrieved.

---

## Key Takeaway

> **PSH (Push) flag in TCP signals that the sender is transmitting actual data** and wants the receiver to pass it to the application immediately, rather than buffering it. In a packet capture, PSH packets with non-zero length are almost always where the interesting payload lives — everything else in a basic TCP exchange is just handshake overhead.
```
