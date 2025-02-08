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
SMB is running on port 139 and 445.

Enum4linux is a tool used to enumerate SMB shares on both Windows and Linux systems.

Command used to conduct a full basic enumeration using Enum4linux:
```bash
enum4linux -a <target-IP>
```
Findings:

Workgroup name: WORKGROUP
![Screenshot (51)](https://github.com/user-attachments/assets/976f9812-a794-4eb3-bbf3-3a20f6dae28a)

Name of the machine: POLOSMB

Operating System version running: 6.1
![Screenshot (53)](https://github.com/user-attachments/assets/6a719092-08de-436e-9134-240333c522a4)

The share that sticks out as something we may want to investigate: profile
![Screenshot (54)](https://github.com/user-attachments/assets/9c159062-a37f-43d7-b52e-7e48599a3dc2)

Why?
1. It is mapped successfully meaning that I have access to it.
2. It allows listing, which means that I can browse its contents.
3. Profiles usually store user data, which when misconfigured, can contain credentials or sensitive files.



