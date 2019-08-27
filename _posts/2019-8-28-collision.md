---
layout: post
title: pwnable.kr - collision
categories: pwn
tags : [Linux, Binary Exploitation, code analysis, Exploit Development, c, Python]
image: pwn/collision/0.png
---

<hr>
### Introduction
#### Hey guys this is my write-up for a challenge called `collision` from [pwnable.kr](http://pwnable.kr/). It's a very simple challenge, we need a password to make the program read the flag, the function that validates the given password is vulnerable to hash collision so we will exploit it.
#### Challenge Description : 
```
Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

ssh col@pwnable.kr -p2222 (pw:guest)
```
![](/images/pwn/collision/0.png)
<hr>
### Code Analysis, Tests
#### `col.c` :
```
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```
#### `main()` :

#### Starting by the main function it checks if we have given the program an input and it checks if our input's length is exactly 20 bytes. Then it checks if the return value of `check_password(our input)` is equal to `hashcode`, if we pass that check it will read the flag, otherwise it will print `wrong passcode.` and exit.
```
int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```
#### Looking up, we can see the declaration of the variable `hashcode`  :
```
unsigned long hashcode = 0x21DD09EC;
```
#### That's a hex value, let's convert it to decimal with python :
```
>>> 0x21DD09EC
568134124
```
#### So we need our input to be 20 bytes length and we also need to make the function `check_password` return `568134124` when our input is given to it.
#### Let's quickly try to simulate that in `gdb`.
#### I ran the program and set a breakpoint at main :
```
gef➤  break main                                                                                                                                                                                         
Breakpoint 1 at 0x11b3                                                                                                                                                                                   
gef➤  r "AAAAAAAAAAAAAAAAAAAA"
Starting program: /root/Desktop/pwnable.kr/collision/col "AAAAAAAAAAAAAAAAAAAA"

Breakpoint 1, 0x00005555555551b3 in main ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00005555555551af  →  <main+0> push rbp
$rbx   : 0x0 
$rcx   : 0x00007ffff7fa9718  →  0x00007ffff7faad80  →  0x0000000000000000
$rdx   : 0x00007fffffffe160  →  0x00007fffffffe48b  →  "SHELL=/bin/bash"
$rsp   : 0x00007fffffffe060  →  0x0000555555555260  →  <__libc_csu_init+0> push r15
$rbp   : 0x00007fffffffe060  →  0x0000555555555260  →  <__libc_csu_init+0> push r15
$rsi   : 0x00007fffffffe148  →  0x00007fffffffe44f  →  "/root/Desktop/pwnable.kr/collision/col"
$rdi   : 0x2                                                                                                           
$rip   : 0x00005555555551b3  →  <main+4> sub rsp, 0x10
$r8    : 0x00007ffff7faad80  →  0x0000000000000000
$r9    : 0x00007ffff7faad80  →  0x0000000000000000
$r10   : 0x0
$r11   : 0x00007ffff7f6b1b0  →  0x0000800003400468
$r12   : 0x0000555555555080  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffe140  →  0x0000000000000002
$r14   : 0x0
$r15   : 0x0
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe060│+0x0000: 0x0000555555555260  →  <__libc_csu_init+0> push r15  ← $rsp, $rbp
0x00007fffffffe068│+0x0008: 0x00007ffff7e1209b  →  <__libc_start_main+235> mov edi, eax
0x00007fffffffe070│+0x0010: 0x0000000000000000
0x00007fffffffe078│+0x0018: 0x00007fffffffe148  →  0x00007fffffffe44f  →  "/root/Desktop/pwnable.kr/collision/col"
0x00007fffffffe080│+0x0020: 0x0000000200040000
0x00007fffffffe088│+0x0028: 0x00005555555551af  →  <main+0> push rbp
0x00007fffffffe090│+0x0030: 0x0000000000000000
0x00007fffffffe098│+0x0038: 0xf6e7f80b45a87e3d
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x5555555551ae <check_password+73> ret
   0x5555555551af <main+0>         push   rbp
   0x5555555551b0 <main+1>         mov    rbp, rsp
 → 0x5555555551b3 <main+4>         sub    rsp, 0x10
   0x5555555551b7 <main+8>         mov    DWORD PTR [rbp-0x4], edi
   0x5555555551ba <main+11>        mov    QWORD PTR [rbp-0x10], rsi
   0x5555555551be <main+15>        cmp    DWORD PTR [rbp-0x4], 0x1
   0x5555555551c2 <main+19>        jg     0x5555555551e6 <main+55>
   0x5555555551c4 <main+21>        mov    rax, QWORD PTR [rbp-0x10]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "col", stopped, reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x5555555551b3 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤
```
#### Then I set a breakpoint before the return instruction in `check_password()` and continued the execution :
```
gef➤  disas check_password
Dump of assembler code for function check_password:
   0x0000555555555165 <+0>:     push   rbp
   0x0000555555555166 <+1>:     mov    rbp,rsp
   0x0000555555555169 <+4>:     mov    QWORD PTR [rbp-0x18],rdi
   0x000055555555516d <+8>:     mov    rax,QWORD PTR [rbp-0x18]
   0x0000555555555171 <+12>:    mov    QWORD PTR [rbp-0x10],rax
   0x0000555555555175 <+16>:    mov    DWORD PTR [rbp-0x8],0x0
   0x000055555555517c <+23>:    mov    DWORD PTR [rbp-0x4],0x0
   0x0000555555555183 <+30>:    jmp    0x5555555551a2 <check_password+61>
   0x0000555555555185 <+32>:    mov    eax,DWORD PTR [rbp-0x4]
   0x0000555555555188 <+35>:    cdqe
   0x000055555555518a <+37>:    lea    rdx,[rax*4+0x0]
   0x0000555555555192 <+45>:    mov    rax,QWORD PTR [rbp-0x10]
   0x0000555555555196 <+49>:    add    rax,rdx
   0x0000555555555199 <+52>:    mov    eax,DWORD PTR [rax]
   0x000055555555519b <+54>:    add    DWORD PTR [rbp-0x8],eax
   0x000055555555519e <+57>:    add    DWORD PTR [rbp-0x4],0x1
   0x00005555555551a2 <+61>:    cmp    DWORD PTR [rbp-0x4],0x4
   0x00005555555551a6 <+65>:    jle    0x555555555185 <check_password+32>
   0x00005555555551a8 <+67>:    mov    eax,DWORD PTR [rbp-0x8]
   0x00005555555551ab <+70>:    cdqe
   0x00005555555551ad <+72>:    pop    rbp
   0x00005555555551ae <+73>:    ret
End of assembler dump.
gef➤  break *0x00005555555551ae
Breakpoint 2 at 0x5555555551ae
gef➤  c
Continuing.

Breakpoint 2, 0x00005555555551ae in check_password ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x46464645
$rbx   : 0x0
$rcx   : 0x6
$rdx   : 0x10
$rsp   : 0x00007fffffffe048  →  0x0000555555555225  →  <main+118> mov rdx, rax
$rbp   : 0x00007fffffffe060  →  0x0000555555555260  →  <__libc_csu_init+0> push r15
$rsi   : 0x00007fffffffe148  →  0x00007fffffffe44f  →  "/root/Desktop/pwnable.kr/collision/col"
$rdi   : 0x00007fffffffe476  →  "AAAAAAAAAAAAAAAAAAAA"
$rip   : 0x00005555555551ae  →  <check_password+73> ret 
$r8    : 0x400
$r9    : 0x00007ffff7faad80  →  0x0000000000000000
$r10   : 0xfffffffffffff479
$r11   : 0x00007ffff7e861e0  →  <__strlen_sse2+0> pxor xmm0, xmm0
$r12   : 0x0000555555555080  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffe140  →  0x0000000000000002
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe048│+0x0000: 0x0000555555555225  →  <main+118> mov rdx, rax       ← $rsp
0x00007fffffffe050│+0x0008: 0x00007fffffffe148  →  0x00007fffffffe44f  →  "/root/Desktop/pwnable.kr/collision/col"
0x00007fffffffe058│+0x0010: 0x0000000200000000
0x00007fffffffe060│+0x0018: 0x0000555555555260  →  <__libc_csu_init+0> push r15  ← $rbp
0x00007fffffffe068│+0x0020: 0x00007ffff7e1209b  →  <__libc_start_main+235> mov edi, eax
0x00007fffffffe070│+0x0028: 0x0000000000000000
0x00007fffffffe078│+0x0030: 0x00007fffffffe148  →  0x00007fffffffe44f  →  "/root/Desktop/pwnable.kr/collision/col"
0x00007fffffffe080│+0x0038: 0x0000000200040000
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x5555555551a1 <check_password+60> add    DWORD PTR [rbx+0x7e04fc7d], eax
   0x5555555551a7 <check_password+66> fisttp QWORD PTR [rbx-0x67b707bb]
   0x5555555551ad <check_password+72> pop    rbp
 → 0x5555555551ae <check_password+73> ret    
   ↳  0x555555555225 <main+118>       mov    rdx, rax
      0x555555555228 <main+121>       mov    rax, QWORD PTR [rip+0x2e19]        # 0x555555558048 <hashcode>
      0x55555555522f <main+128>       cmp    rdx, rax
      0x555555555232 <main+131>       jne    0x55555555524c <main+157>
      0x555555555234 <main+133>       lea    rdi, [rip+0xe08]        # 0x555555556043
      0x55555555523b <main+140>       mov    eax, 0x0
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "col", stopped, reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x5555555551ae → check_password()
[#1] 0x555555555225 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤
```
#### The return value of `check_password()` is saved in `EAX` we need it to be `568134124` :
```
gef➤  print $eax
$1 = 0x46464645
gef➤  set $eax=568134124
```
#### Now if we continue execution it should attempt to execute `/bin/cat flag` :
```
gef➤  c
Continuing.
[Detaching after fork from child process 3166]
/bin/cat: flag: No such file or directory
[Inferior 1 (process 3101) exited normally]
gef➤ 
```
#### Great, now we need to find out how to make `check_password()` return that value, let's look at the code.
#### `check_password()` :
```
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}
```
#### This function casts the given passcode (`p`) into integer, declares `ip` which is an [array of pointers](https://www.tutorialspoint.com/cprogramming/c_array_of_pointers.htm) starting with the pointer to `p`, and declares an `int` variable called `res` and gives it a value of `0` then it loops 5 times through `ip` (because length of `passcode` is 20, `20/4 == 5`) and adds each value to `res`, finally it returns `res`.
#### In case you're confused, simply what happens is that it takes the given passcode which is 20 bytes length and divides it to 5 pieces (each piece 4 bytes) then it sums the decimal value of the 5 pieces and returns that value. For example the result of giving `check_password()` "AAAAAAAAAAAAAAAAAAAA" will be like this :
```
"AAAA" + "AAAA" + "AAAA" + "AAAA" + "AAAA"
0x41414141 + 0x41414141 + 0x41414141 + 0x41414141 + 0x41414141
1094795585 + 1094795585 + 1094795585 + 1094795585 + 1094795585
res = 5473977925
```
#### To verify that I took the main code and added some `printf` statements, test code looks like this :
```
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
		printf("\n--------------------------\n");
		printf("loop :  %i\n", i);
		printf("piece value : %i\n",ip[i] );
		printf("\n");
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}
	printf("hashcode : %i\n", hashcode);
	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```
#### Let's give it 20 A's :
```
root@kali:~/Desktop/pwnable.kr/collision/test# ./test "AAAAAAAAAAAAAAAAAAAA"
hashcode : 568134124

--------------------------
loop :  0
piece value : 1094795585


--------------------------
loop :  1
piece value : 1094795585


--------------------------
loop :  2
piece value : 1094795585


--------------------------
loop :  3
piece value : 1094795585


--------------------------
loop :  4
piece value : 1094795585

wrong passcode.

```
#### You can see that the 5 pieces are of the same value which is the value of `0x41414141` (4 A's) :
```
>>> 0x41414141
1094795585
```
#### And if we give it `AAAABBBBCCCCDDDDEEEE` :
```
root@kali:~/Desktop/pwnable.kr/collision/test# ./test "AAAABBBBCCCCDDDDEEEE"         
hashcode : 568134124

--------------------------
loop :  0
piece value : 1094795585


--------------------------
loop :  1
piece value : 1111638594


--------------------------
loop :  2
piece value : 1128481603


--------------------------
loop :  3
piece value : 1145324612


--------------------------
loop :  4
piece value : 1162167621

wrong passcode.

```

```
>>> 0x41414141
1094795585
>>> 0x42424242
1111638594
>>> 0x43434343
1128481603
>>> 0x44444444
1145324612
>>> 0x45454545
1162167621
```
<br>
<hr>
### Exploitation
#### We need to come up with 5 pieces that add up to 568134124. 
#### We can divide the original value by 5 :
```
>>> 568134124/5
113626824
```
#### But 568134124 isn't divisible by 5 :
```
>>> 568134124%5
4
```
#### We can use 113626824 as the first 4 pieces, to get the last piece we will multiply 113626824 by 4 and subtract the result from 568134124 :
```
>>> 113626824 * 4
454507296
>>> 568134124 - 454507296
113626828
```
#### What's left is to convert them to hex :
```
>>> hex(113626824)
'0x6c5cec8'
>>> hex(113626828)
'0x6c5cecc'
```
#### And because it's little endian we will reverse the order, final payload will be :
```
python -c 'print "\xc8\xce\xc5\x06" * 4 + "\xcc\xce\xc5\x06"'
```
#### Let's test it :
```
root@kali:~/Desktop/pwnable.kr/collision/test# ./test `python -c 'print "\xc8\xce\xc5\x06" * 4 + "\xcc\xce\xc5\x06"'`                                                                 
ip : 1329747125

--------------------------
loop :  0
piece value : 113626824

--------------------------
loop :  1
piece value : 113626824

--------------------------
loop :  2
piece value : 113626824

--------------------------
loop :  3
piece value : 113626824

--------------------------
loop :  4
piece value : 113626828

/bin/cat: flag: No such file or directory
```
#### It works.
![](/images/pwn/collision/1.png)
#### And by the way what we did now is a [hash collision](https://learncryptography.com/hash-functions/hash-collision-attack), we made a hash function produce the same output for different inputs.
#### I also wrote small python script using `pwntools` :
```
#!/usr/bin/python
from pwn import *

payload = p32(0x6c5cec8) * 4 + p32(0x6c5cecc)

r = ssh('col' ,'pwnable.kr' ,password='guest', port=2222)
p = r.process(executable='./col', argv=['col',payload])
flag = p.recv()
log.success("Flag: " + flag)
p.close()
r.close()
```
![](/images/pwn/collision/2.png)
#### pwned !
#### That's it , Feedback is appreciated !
#### Don't forget to read the [other write-ups](/categories) , Tweet about the write-up if you liked it , follow on twitter for awesome resources [@Ahm3d_H3sham](https://twitter.com/Ahm3d_H3sham)
#### Thanks for reading.
#### Previous pwn write-up : [pwnable.kr - bof](/pwn/bof/)
<br>
<hr>