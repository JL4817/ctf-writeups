# Snapped Phish-ing Line

## Challenge Overview

The challenge provides a **virtual environment with Thunderbird** containing a set of phishing emails.
The goal is to investigate the emails, analyse attachments, and track down a hidden flag.

---

## Exploitation

### Step 1 — Analyse Emails in Thunderbird

Opening Thunderbird reveals several phishing emails. Most questions can be answered by browsing through the emails and their headers directly in the client.

### Step 2 — Find the Redirection URL (Zoe Duncan's Email)

Opening the attachment from the email addressed to **Zoe Duncan** in a text editor reveals a hardcoded redirection URL inside the file. Extracting the root domain from that URL answers the question.

### Step 3 — SHA256 Hash of the File

We extract the attachment and compute its hash:

```bash
sha256sum <extracted_file>
```

Alternatively, upload the file to **CyberChef** with:
- Operation: `SHA2`
- Size: `256`
- Rounds: `64`

```
ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```

### Step 4 — VirusTotal Analysis

We look up the hash on VirusTotal:

```
https://www.virustotal.com/gui/file/ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```

The ZIP archive is flagged under two threat categories — **phishing** and one additional category found in the VirusTotal report.

### Step 5 — Find the Flag

The challenge hint states the flag is in a file named `flag.txt`. Based on the redirection URL found in Step 2, we construct the path to the flag by navigating the phishing site's directory structure:

```
kennaroads.buzz/data/Update365/office365/flag.txt
```

The page returns the flag encoded in **Base64**. Decoding it reveals the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **Phishing sites often host additional malicious payloads in predictable directory structures.** Once a root domain is identified, enumerating common paths (e.g. `/data/`, `/office365/`) can expose further attacker infrastructure — and in this case, the flag itself.
```
