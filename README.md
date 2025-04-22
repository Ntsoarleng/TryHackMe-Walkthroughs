# TryHackMe-Walkthrough

# Network Services Analysis 🔍

This repository contains an analysis of network services, including enumeration, authentication testing, and service exploitation techniques.

## 📖 Documentation
All detailed findings, commands, and methodologies are documented in [`NetworkServices.md`](./NetworkServices.md).

## 🔹 Overview
This project involves identifying and analyzing open ports, checking service versions, and testing authentication mechanisms for potential vulnerabilities.

## 🚀 Tools & Techniques Used
The following tools and methodologies were used for enumeration, service analysis, and exploitation:

- **Nmap** – Scanning and enumerating open ports and services.
- **Enum4linux** – Enumerating SMB shares and user accounts.
- **Telnet** – Checking for accessible Telnet sessions.
- **FTP** – Accessing and analyzing anonymous FTP servers.
- **Hydra** – Performing brute-force attacks on authentication services.
- **Netcat (nc)** – Establishing reverse shells and network communication.
- **Msfvenom** – Generating payloads for command execution.

## 🔥 Key Areas Covered
- **Port scanning and service enumeration**
- **Analyzing FTP, SMB, and Telnet access**
- **Brute-force authentication testing**
- **Payload creation and execution**

## 🛠 How to Use
1. Read [`NetworkServices.md`](./NetworkServices.md) for full details.
2. Use the provided commands and techniques in a controlled environment.
3. Experiment responsibly and follow ethical guidelines.

## 🌍 TryHackMe Room
This project is based on the **TryHackMe** room: [Network Services](https://tryhackme.com/room/networkservices).  


### 📘 Network Services 2 – More Common Services & Misconfigurations

This walkthrough explores additional vulnerable network services beyond those in Network Services 1. I focused on enumerating and exploiting services like **NFS**, **SMTP**, and **SQL**, showcasing how common misconfigurations can lead to serious security risks.

🔍 **Key skills demonstrated:**
- Enumerating and mounting NFS shares  
- Probing SMTP with VRFY/EXPN for user discovery  
- Crafting and sending spoofed emails via SMTP  
- Enumerating and exploiting SQL vulnerabilities  

📂 Full walkthrough: [`Network Services 2`](./NetworkServices2.md)
## 🌍 TryHackMe Room
This project is based on the **TryHackMe** room: [Network Services 2](https://tryhackme.com/room/networkservices2).  

---

🙋‍♀️ Author: Made by Ntsoareleng

📅 Year: 2025

🌱 Learning Python, cybersecurity, and cloud — one project at a time!

