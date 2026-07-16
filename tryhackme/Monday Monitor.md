# Monday Monitor

## Challenge Overview

The challenge provides a **Wazuh/Elastic SIEM environment** with endpoint logs from Swiftspend Finance.
The goal is to investigate suspicious activity recorded on **Apr 29, 2024 between 12:00 and 20:00**.

Access the dashboard at `https://LAB_WEB_URL.p.thmlabs.com` with credentials `admin / Mond*yM0nit0r7`, then navigate to **Security Events** and load the saved query `Monday_Monitor`.

---

## Exploitation

### Step 1 — Find the Initial Access File

Search the logs for:
```
localhost
```

Inspect `data.win.eventdata.commandLine` in the returned events. The downloaded file used for initial access appears as:

```
SwiftSpend_Financial_Expenses.xlsm
```

✅ **Answer:** `SwiftSpend_Financial_Expenses.xlsm`

### Step 2 — Find the Scheduled Task Command

Search for:
```
schtasks
```

Inspect `data.win.eventdata.parentCommandLine`. The full command is:

```
"cmd.exe" /c "reg add HKCU\SOFTWARE\ATOMIC-T1053.005 /v test /t REG_SZ /d cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0= /f & schtasks.exe /Create /F /TN "ATOMIC-T1053.005" /TR "cmd /c start /min "" powershell.exe -Command IEX([System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String((Get-ItemProperty -Path HKCU:\\SOFTWARE\\ATOMIC-T1053.005).test)))" /sc daily /st 12:34"
```

✅ **Answer:** Full command above.

### Step 3 — Find the Scheduled Task Time

From the same event, the `/st` flag specifies the run time:

```
/st 12:34
```

✅ **Answer:** `12:34`

### Step 4 — Decode the Encoded String

The command stores a Base64 string in the registry before the scheduled task decodes and executes it:

```
cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0=
```

Decoding it:

```bash
echo "cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0=" | base64 -d
# ping www.youarevulnerable.thm
```

✅ **Answer:** `ping www.youarevulnerable.thm`

### Step 5 — Find the New User Password

Search for:
```
net
```

Inspect `data.win.eventdata.parentCommandLine` to find the `net user` command that created a new account, which includes the plaintext password.

### Step 6 — Find the Credential Dumping Tool

Search for:
```
mimikatz
```

Inspect `data.win.eventdata.Image` to find the `.exe` filename used to dump credentials.

### Step 7 — Find the Exfiltrated Flag

Search for:
```
THM
```

Inspect `data.win.eventdata.CommandLine` to find the command that exfiltrated data containing the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **Scheduled tasks combined with registry-stored Base64 payloads are a classic persistence technique (MITRE T1053.005).** The attacker stored the malicious command encoded in the registry to avoid plaintext detection, then used a scheduled task to decode and execute it via PowerShell. SIEM tools like Wazuh can catch this by monitoring `schtasks`, `reg add`, and encoded PowerShell invocations.
```
