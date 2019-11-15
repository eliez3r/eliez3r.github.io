---
title: "[pwnable.kr]Toddler/flag"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180914
modify_date: 2018-09-14
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

```
Papa brought me a packed present! let's open it.

Download : http://pwnable.kr/bin/flag

This is reversing task. all you need is binary
```

이번 문제는 소스코드가 없고 바이너리 하나를 던져주고 리버싱으로 풀어야 한다.

-----

### 0x00. Analysis

```
root@ubuntu:~/pwnable/flag# checksec flag
[*] '/root/pwnable/flag/flag'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
    Packer:   Packed with UPX
```

파일은 다운받아 `checksec`을 돌려보면 64bit 바이너리이며, UPX로 실행압축되어 있음을 알 수 있다.

```
root@ubuntu:~/pwnable/flag# ./flag
I will malloc() and strcpy the flag there. take it.
```

바이너리를 실행해보면 "malloc하고 strcpy로 flag를 거기에 복사한다." 뭐 이런 이야기인거 같다. ~~영어 못하니까 패스~~!

일단 UPX압축을 풀어보자.

```
root@ubuntu:~/pwnable/flag# upx -d flag -o un_flag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2018
UPX 3.95        Markus Oberhumer, Laszlo Molnar & John Reiser   Aug 26th 2018

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    883745 <-    335288   37.94%   linux/amd64   un_flag

Unpacked 1 file.
```

```
root@ubuntu:~/pwnable/flag# checksec un_flag
[*] '/root/pwnable/flag/un_flag'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

정상적으로 잘 압축이 풀렸다.

이제 gdb로 까보자.

```
gdb-peda$ pdisas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:     push   rbp
   0x0000000000401165 <+1>:     mov    rbp,rsp
   0x0000000000401168 <+4>:     sub    rsp,0x10
   0x000000000040116c <+8>:     mov    edi,0x496658
   0x0000000000401171 <+13>:    call   0x402080 <puts>
   0x0000000000401176 <+18>:    mov    edi,0x64
   0x000000000040117b <+23>:    call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401184 <+32>:    mov    rdx,QWORD PTR [rip+0x2c0ee5]
   0x000000000040118b <+39>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:    mov    rsi,rdx
   0x0000000000401192 <+46>:    mov    rdi,rax
   0x0000000000401195 <+49>:    call   0x400320
   0x000000000040119a <+54>:    mov    eax,0x0
   0x000000000040119f <+59>:    leave
   0x00000000004011a0 <+60>:    ret
End of assembler dump.
```

`main+8(mov edi,0x496658)` 부분에 `0x496658` 주소를 `edi`에 넣고 puts를 실행하는데 아마 `0x496658`에는 아까 출력되었던 "I will malloc() and strcpy the flag there. take it." 문자열이 들어있을 것이다. (확인사살)

```
gdb-peda$ x/s 0x496658
0x496658:       "I will malloc() and strcpy the flag there. take it."
```

그리고 `main+18(mov edi,0x64)` 부터 0x64(100) 만큼 malloc을 하고 그 주소 값을 `rbp-0x8`에 넣는 것을 볼수 있다.

마지막으로 호출되는 `0x400320`은 정황상 strcpy일것이라고 추축하고 그 앞을 보면, `rsi`에서 `rdi`로 복사가 이뤄지는데, `rdi`는 malloc한 주소 값이 들어가고,`rsi`에는 `rip+0x2c0ee5` 가 복사된다. 이 부분이 flag일 가능성이 크다.

따라서 `rdx`에 복사되는 `main+32(mov rdx,QWORD PTR [rip+0x2c0ee5])` 부분을 실행하고 나서 `rdx`를 살펴보면 flag가 있을 것이다.

```
[-------------------------------------code-------------------------------------]
   0x40117b <main+23>:  call   0x4099d0 <malloc>
   0x401180 <main+28>:  mov    QWORD PTR [rbp-0x8],rax
   0x401184 <main+32>:  mov    rdx,QWORD PTR [rip+0x2c0ee5]
=> 0x40118b <main+39>:  mov    rax,QWORD PTR [rbp-0x8]
   0x40118f <main+43>:  mov    rsi,rdx
   0x401192 <main+46>:  mov    rdi,rax
   0x401195 <main+49>:  call   0x400320
   0x40119a <main+54>:  mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe3d0 --> 0x401a50 (<__libc_csu_init>:   push   r14)
0008| 0x7fffffffe3d8 --> 0x6c96b0 --> 0x0
0016| 0x7fffffffe3e0 --> 0x0
0024| 0x7fffffffe3e8 --> 0x401344 (<__libc_start_main+404>:     mov    edi,eax)
0032| 0x7fffffffe3f0 --> 0x0
0040| 0x7fffffffe3f8 --> 0x100000000
0048| 0x7fffffffe400 --> 0x7fffffffe4d8 --> 0x7fffffffe735 ("/root/pwnable/flag/un_flag")
0056| 0x7fffffffe408 --> 0x401164 (<main>:      push   rbp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x000000000040118b in main ()
gdb-peda$ x/s $rdx
0x496628:       "UPX...? sounds like a delivery service :)"
```

-----

### 0x01. Exploit

끄읏 :)

-----

