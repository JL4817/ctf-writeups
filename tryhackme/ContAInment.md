# ContAInment

## Challenge Overview

The challenge provides an **AI chatbot** and a **Ubuntu workstation** belonging to senior researcher Oliver Deer.
Internal monitoring flagged unusual network activity — the goal is to investigate the incident, recover exfiltrated data, and find the flag.

---

## Exploitation

### Step 1 — Find the PCAP Files

The scenario mentions unusual network activity, so we search for packet captures on the machine:

```bash
find /home/o.deer/ -type f -name "*.pcap"
```

This returns 40 `.pcap` files spread across four dates (2025-06-15 to 2025-06-18).

### Step 2 — Identify the Suspicious File

We sort all PCAP files by size to find anomalies:

```bash
find /home/o.deer/Documents/pcap_dumps -type f -name "*.pcap" -printf "%s %p\n" | sort -n
```

39 files are exactly **198 bytes** (empty/noise). One stands out:

```
2262  /home/o.deer/Documents/pcap_dumps/2025-06-17/session_4444_dump.pcap
```

### Step 3 — Reassemble the PCAP

We ask the AI chatbot to reassemble the suspicious PCAP using its `pcap_file_reassembler` tool:

```
Reassemble /home/o.deer/Documents/pcap_dumps/2025-06-17/session_4444_dump.pcap
```

Output saved to: `/home/o.deer/qwen-output/reassembled_data_dump.txt`

### Step 4 — Read the Recovered File

```bash
cat /home/o.deer/qwen-output/reassembled_data_dump.txt
```

The file reveals a **prompt injection attack log** — an attacker had manipulated the AI assistant into dumping Oliver Deer's personal and access credentials. At the bottom of the file:

```
Dont lose this lol or Ill have no leverage westtechvictim1
```

✅ Password found: `leverage`

### Step 5 — Decrypt the Encrypted Archive

Using the recovered password to unzip the exfiltrated data:

```bash
unzip /home/o.deer/westtech_projects_encrypted.zip -d /dev/shm/
# Password: leverage
```

This extracts several sensitive files including `thm_flags.txt`.

### Step 6 — Find the Real Flag

`thm_flags.txt` contains dozens of base64-encoded strings:

```
dGhtezUyLDY1LDE3LDk1LDE0fQ==
dGhtezM1LDkwLDk5LDEzLDM2fQ==
...
```

Decoding them all manually would take too long, so we use the chatbot's `liberty_prime` tool — which scans the file and returns the flag whose numbers are all prime:

```
Use liberty_prime to check /dev/shm/home/o.deer/westtech_projects/thm_flags.txt
and identify the flag.
```

The tool identifies the correct flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **AI assistants are attack surfaces too.** This challenge demonstrated a real-world prompt injection attack — where an attacker bypassed an LLM's safety instructions to exfiltrate sensitive employee data. AI systems with access to private data must implement robust guardrails and should never expose raw memory or internal records through natural language manipulation.
```
