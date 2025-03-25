# Network Services 2  
🔍 **Enumerating and Exploiting More Common Network Services & Misconfigurations**  

📅 **Date:** 2025-03-22

🖥️ **Category:** Network Services, Penetration Testing  
🎯 **Objective:** Identify and exploit misconfigurations in common network services  


## 🛠 1️⃣ Enumerating Network File System (NFS)  
### 🔎 **Scanning for Open Ports**  

The first step of enumeration is to conduct a port scan to find out as much information as we can about the services, open ports and operating system of the target machine.

I am going to scan for open ports using an nmap scan.

**Command used:**
```bash
nmap -p- --open <target-IP>
```

**Findings:**

![Screenshot (143)](https://github.com/user-attachments/assets/67458606-c6c6-4c28-9a49-4fb278b6ef65)

1. There are 7 open ports.
2. Port 2049 contains the service we want to enumerate.

Now I used the command below to list the NFS shares

**Command used:**
```bash
/usr/sbin/showmount -e <target-IP>
```

**Findings:**

![Screenshot (144)](https://github.com/user-attachments/assets/269a3670-f6e5-4265-bb7c-6120b2d8685a)

1. The name of the visible share found is /home.

Now we want to mount the share to our local machine. Firstly, I make create a directory that I will mount my share to using *mkdir /tmp/mount*. My client system needs a directory where all the content shared by the host server in the export folder can be accessed, and we have created this directory, it is called tmp.

I then mounted the NFS share to my local machine.

**Command used:**
```bash
sudo mount -t nfs 10.10.10.71:/home /tmp/mount/ -nolock
```
I then changed directory to where I mounted this share.

**Findings:**

![Screenshot (145)](https://github.com/user-attachments/assets/00208cc0-df51-40aa-94cb-73b4320f10d2)

1. The name of the folder inside this directory is cappucino.

![Screenshot (146)](https://github.com/user-attachments/assets/4d91018c-9aff-478f-8e0a-8c74180a7ee7)

I took a look insde this directory and looked at the files. It looks like we are inside a user's home directory.



