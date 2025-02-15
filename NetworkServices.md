# TryHackMe Walkthrough: Network Services

📅 Date: 2025-02-04  
🔍 Difficulty Level: Easy 

🖥️ Category: Enumeration, Exploitation.  
🎯 Objective: Learn about, then enumerate and exploit a variety of network services and misconfigurations.

## 🛠1️⃣ **Enumerating SMB**
### 🔎 **Network Scanning**
**Command used:**
```bash
nmap -p- --open <target-IP>
```
**Findings:**
The scan revealed the following three open ports,
1. 22/tcp (ssh)
2. 139/tcp (netbios-ssn)
3. 445/tcp (microsoft-ds)
   
SMB is running on port 139 and 445.


Enum4linux is a tool used to enumerate SMB shares on both Windows and Linux systems.

**Command used** to conduct a full basic enumeration using Enum4linux:
```bash
enum4linux -a <target-IP>
```
**Findings:**

Workgroup name: WORKGROUP

![Screenshot (51)](https://github.com/user-attachments/assets/976f9812-a794-4eb3-bbf3-3a20f6dae28a)

Name of the machine: POLOSMB

Operating System version running: 6.1

![Screenshot (53)](https://github.com/user-attachments/assets/6a719092-08de-436e-9134-240333c522a4)

The share that sticks out as something we may want to investigate: profiles

![Screenshot (54)](https://github.com/user-attachments/assets/9c159062-a37f-43d7-b52e-7e48599a3dc2)

Why?
1. It is mapped successfully meaning that I have access to it.
2. It allows listing, which means that I can browse its contents.
3. Profiles usually store user data, which when misconfigured, can contain credentials or sensitive files.



## 📂2️⃣ **Exploiting SMB**

We're going to be exploiting anonymous SMB share access- a common misconfiguration that can allow us to gain information that will lead to a shell. 
We want to see if the profiles share does not require authentication to view the files.

**Command used:**
```bash
smbclient //<target-IP>/profiles -U Anonymous -p 445
```

**Findings:**

The share allows for anonymous access without providing a password. I also used the ls command to look for interesting documents that could contain valuable information.

![Screenshot (58)](https://github.com/user-attachments/assets/cd4f49c6-ca0b-4976-93ed-5355e41580a5)

The "Working From Home Information.txt" is the document that appeared interesting and took a look into it by using the command below:

```bash
smb: \> more "Working From Home Information.txt"
```

![Screenshot (57)](https://github.com/user-attachments/assets/d40f3292-be0d-407a-9d9d-42816e346d1d)

We can assume that this profile folder belongs to John Cactus. The ssh service has been configured to allow him to work from home meaning that the directory in the share that we should look into is .ssh.

The .ssh directory has a directory that contains authentication keys that allow a user to authenticate themselves on, and access a server. Of these keys, the one that is most useful to us is: id_rsa

I got to this directory by changing to the .ssh directory using the *cd .ssh* command then listed the contents in it using *ls*.

![Screenshot (60)](https://github.com/user-attachments/assets/b82501fd-d32c-48c6-ad30-963f2b34515e)

Now I will be using all the information I gathered to work out the username of the account. Then, use the service and key to log-in to the server.

Firstly, I had to download the id_rsa file into my local machine. I did that by using the command below:

```bash
smb: \> get .ssh/id_rsa id_rsa
```
I then used an exit command to return to my root directory.

I used the *ls* command to see if the id_rsa file had downloaded successfully in my local machine and indeed it had.

![Screenshot (61)](https://github.com/user-attachments/assets/100693d3-2a44-421d-af66-3c6630197d9c)

I then used the *ls -la* command to look list the files in the root directory and their permissions. 

![Screenshot (62)](https://github.com/user-attachments/assets/614e460f-f0e7-405d-bffd-c9c45c5e82e9)


I however changed the id_rsa permissions to 600 using:

```bash
chmod 600 id_rsa
```

This changes the file permissions to allow only the owner to read and write the file. 600 ensures secure SSH key usage, preventing other users from reading or modifying it.

After working out the username of the account, I entered the command below to direct me there. I then received a propt asking me if I was sure that I wanted to continue connection, said yes then I was in.

```bash
ssh -i id_rsa cactus@<target-IP> 
```

I then used the service and key to log in to the server.

![Screenshot (64)](https://github.com/user-attachments/assets/ddf28bd6-2a3b-40cc-9c64-d067e7b4a641)



## 🔌 3️⃣ **Enumerating Telnet** 



