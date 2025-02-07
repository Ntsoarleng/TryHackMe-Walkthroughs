# TryHackMe Walkthrough: Network Services
📅 Date: 2025-02-04  
🔍 Difficulty Level: Easy

🖥️ Category: Enumeration, Exploitation.  
🎯 Objective: Learn about, then enumerate and exploit a variety of network services and misconfigurations.

## 🛠1️⃣ **Enumerating SMB**
### 🔎 **Network Scanning**
Command used:
```bash
nmap -p- --open <target-IP>
```
Findings: 
The scan revealed the following three open ports,
1. 22/tcp (ssh)
2. 139/tcp (netbios-ssn)
3. 445/tcp (microsoft-ds)

