---
title: "PLT와 GOT를 파헤쳐보자"
tags: [plt, got, static link, dynamic link, linking]
author: eli_ez3r
key: 20191115
modify_date: 2019-11-15
article_header:
  type: cover
  image:
    src:
---

## PLT (Procedure Linkage Table)

- 외부 프로시저를 연결시켜주는 테이블
- PLT를 통해 다른 라이브러리에 있는 프로시저를 호출해 사용할 수 있다.

-----

## GOT (Global Offset Table)

- PLT가 참조하는 테이블
- 프로시저들의 주소가 들어있다.



##### 함수를 호출하면(PLT를 호출하면) GOT로 점프하는데 GOT에는 실제 함수주소가 쓰여있다.

- 첫 번째 호출이라면 GOT는 함수의 주소를 가지고 있지 않고 ‘어떤 과정’을 거쳐 주소를 알아낸다.
- 두 번째 호출부터는 첫번째 호출 때 알아낸 주소로 바로 점프한다.

>의문점 1. 함수 주소로 바로 점프하면 되지 왜 PLT와 GOT를 두고 사용하는 것일까?
>
>의문점 2. GOT가 함수의 주소를 알아내는 과정이라는 건 뭘까?

-----

### STATIC LINK와 DYNAMIC LINK : GOT와 PLT는 왜 사용할까?

어떤 코드를 작성한다고 생각해 보자. 

예를들어, 소스안에는 printf함수를 호출하는 코드가 있고 include 한 헤더파일에는 printf의 선언이 있다. 소스파일을 실행파일로 만들기 위해서 아래와 같은 ‘컴파일(compile)’ 과정을 거친다.

![GNU C 컴파일 과정](http://eliez3r.synology.me/assets/img/study/system/plt and got/1.png)

컴파일을 통해 오브젝트 파일이 생성된다. 하지만 오브젝트파일은 그 자체로 실행 불가능하다. 이유는 printf의 구현 코드를 모르기 때문이다. printf를 호출 했을 때 어떤 코드를 실행해야 하는지 우리가 작성한 코드만 가지고서는 아무것도 알 수 없다.

오브젝트 파일을 실행 가능하게 만들기 위해서는 printf의 실행 코드를 찾아서 오브젝트 파일과 연결시켜 주어야 한다. printf의 실행코드는 printf의 구현 코드를 컴파일한 오브젝트 파일로, 이런 오브젝트 파일들이 모여있는 곳을 **라이브러리(Library)**라고 한다.

![소스파일이 실행파일이 되기까지](http://eliez3r.synology.me/assets/img/study/system/plt and got/2.png)

이렇게 라이브러리 등 필요한 **오브젝트 파일들을 연결시키는 작업을 링킹(Linking)**이라고 한다.
이렇게 링크 과정까지 마치면 최종적인 실행파일이 생기게 된다.

링크(Link)를 하는 방법에는 Static 방식과 Dynamic 방식이 있다.

-----

![Static Link방식을 통한 실행파일 생성](http://eliez3r.synology.me/assets/img/study/system/plt and got/3.png)

**Static Link 방식**은 파일 생성시 라이브러리 내용을 포함한 실행파일을 만든다. gcc옵션 중 `static`옵션을 적용하면 Static Link 방식으로 컴파일 된다.

```
root@kali:~/Desktop/BoB7/study# gcc -o test test.c -static
root@kali:~/Desktop/BoB7/study# file test
test: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=d28b83f000e862ee5c91bc63b07ae115ab3d17e2, not stripped
```

실행 파일 안에 모든 코드가 포함되기 때문에 라이브러리 연동 과정이 따로 필요없고, 한번 생성한 파일에 대해서 필요한 라이브러리를 따로 관리하지 않아도 되기 때문에 편하다는 장점이 있다. 하지만 파일 크기가 커지는 단점이 있고 동일한 라이브러리를 사용하더라도 해당 라이브러리를 사용하는 모든 프로그램들은 라이브러리의 내용을 메모리에 맵핑시켜야 한다.

-----

![dynamic linking](http://eliez3r.synology.me/assets/img/study/system/plt and got/4.png)

**Dynamic Link 방식**은 공유 라이브러리를 사용한다. 라이브러리를 하나의 메모리 공간에 맵핑하고 여러 프로그램에서 공유하여 사용하게 된다.

실행파일 안에 라이브러리 코드를 포함하지 않으므로 Static Link 방식을 사용해 컴파일 했을 때에 비해 파일 크기가 훨씬 작아진다. 실행시에도 상대적으로 적은 메모리를 차지한다. 또한 라이브러리를 업데이트 할 수 있기 때문에 유연한 방법이다. 하지만 실행파일이 라이브러리에 의존하기 때문에 라이브러리가 없으면 실행 불가능하다.

```
root@kali:~/Desktop/BoB7/study# gcc -o test test.c
root@kali:~/Desktop/BoB7/study# file test
test: ELF 64-bit LSB pie executable x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=6bbcd278a8fe334d5c6bb2da4b19638baa7530f4, not stripped
```

> gcc는 디폴트로 Dynamic Link방식을 사용한다.

이제 본론으로 돌아가서 Dynamic Link방식으로 컴파일 했을 때 PLT와 GOT를 사용한다고 하였다. 그 이유에 대해서 알아보자.

-----

Static Link 방식으로 컴파일 하면 라이브러리가 프로그램 내부에 있기 때문에 함수의 주소를 알아오는 과정이 필요 없지만, **Dynamic Link 방식으로 컴파일 하면 라이브러리가 프로그램 외부에 존재하기 때문에 함수의 주소를 알아오는 과정이 필요하다.**

![PLT와 GOT의 호출관계](http://eliez3r.synology.me/assets/img/study/system/plt and got/5.png)

Dynamic Link 방식으로 프로그램이 만들어지면 함수를 호출 할 때 **PLT**를 참조하게 된다. PLT에서는 **GOT로 점프**를 하는데, **GOT에 라이브러리에 존재하는 실제 함수의 주소가 쓰여있어서 함수를 호출**하게 된다.

하지만, 첫번째 호출이라면 GOT에 실제 함수의 주소가 있지 않다. 그래서 **첫 호출 시에는 Linker가 `dl_resolve`라는 함수를 사용해 필요한 함수의 주소를 알아오고, GOT에 그 주소를 써준 후 해당 함수를 호출한다.**

```
root@kali:~/study# gdb -q ./test
Reading symbols from ./test...(no debugging symbols found)...done.
gdb-peda$ set disassembly-flavor intel
gdb-peda$ pdisas main
Dump of assembler code for function main:
   0x000000000000064a <+0>:	push   rbp
   0x000000000000064b <+1>:	mov    rbp,rsp
   0x000000000000064e <+4>:	mov    edi,0x41
   0x0000000000000653 <+9>:	call   0x520 <putchar@plt>
   0x0000000000000658 <+14>:	mov    eax,0x0
   0x000000000000065d <+19>:	pop    rbp
   0x000000000000065e <+20>:	ret
End of assembler dump.
gdb-peda$ p putchar
$1 = {<text variable, no debug info>} 0x520 <putchar@plt>	<<<<<<<<<<<<<<
gdb-peda$ r
Starting program: /root/Desktop/BoB7/study/test
A[Inferior 1 (process 24523) exited normally]
Warning: not running or target is remote
gdb-peda$ p putchar
$2 = {int (int)} 0x7ffff7a8f200 <putchar>			<<<<<<<<<<<<<<
```

putchar의 함수 호출 전과 후가 다르고, 두 번째 호출부터 putchar함수의 주소를 가리키고 있다. 

그렇다면 GOT에 필요한 함수의 주소는 어떻게 쓰여지게 되는 걸까?

> 64bit 환경에서 gcc를 이용하여 32bit 컴파일 하기
> gcc에서 **-m32** 옵션을 주면 되는데, 해당 옵션을 주기 위해서는 다음 패키지를 설치해야 한다.
> **apt-get install gcc-multilib**

> 칼리리눅스 최신버전에서는 gcc 컴파일시 자동으로 PIE옵션이 enable되어 있다. PIE는 코드영역 주소를 랜덤화 시키는 보호기법이다. PIE를 disable하는 방법은 gcc 이용할 때 **-no-pie** 옵션을 넣어주면 된다.

```
root@kali:~/study# gcc -m32 -no-pie test.c -o test
```

-----

이제 본격적으로 함수의 호출 과정을 살펴보자.

```
=> 0x8048446 <main+32>:	call   0x80482f0 <putchar@plt>
```

putchar가 호출되어 지는데 해당 주소로 가보자.

```
gdb-peda$ x/3i 0x080482f0
   0x80482f0 <putchar@plt>:	jmp    DWORD PTR ds:0x804a010
   0x80482f6 <putchar@plt+6>:	push   0x8
   0x80482fb <putchar@plt+11>:	jmp    0x80482d0
```

puchar는 PLT를 가리키고 있다. PLT에서는 GOT을 참조한다고 했으니, 이 0x0804a010은 GOT일 것이다. 확인해보자.

```
gdb-peda$ x/x 0x0804a010
0x804a010 <putchar@got.plt>:	0x080482f6      // plt+6의 주소
```

호출 관계를 정리해보면, 함수가 처음 호출되면 plt+6을 실행한다. plt+6에서는 0x8을 PUSH하고 또 0x80482d0 주소로 점프하게 된다. 여기서부터 Dynamic Linking의 시작이다.

```
gdb-peda$ x/2i 0x080482d0
   0x80482d0:	push   DWORD PTR ds:0x804a004
   0x80482d6:	jmp    DWORD PTR ds:0x804a008
```

```
gdb-peda$ x/x 0x804a004
0x804a004:	0xf7ffd940       // link_map 구조체 포인터
```

```
gdb-peda$ x/x 0x804a008
0x804a008:	0xf7fead80

gdb-peda$ x/i 0xf7fead80
   0xf7fead80 <_dl_runtime_resolve>:	push   eax
```

그 주소로 점프했더니 또 어떤 값을 PUSH하고 다른 곳으로 점프한다. 여기서 PUSH되는 값(0xf7ffd940)은 l**ink_map 구조체 포인터**이다. 

linke_map 구조체는 말 그대로 ld loader가 참조하는 링크 지도로, 라이브러리의 정보를 담고 있다. 이 link_map 구조체를 통해 여러 가지 테이블의 주소를 구할 수 있다.

link_map구조체를 PUSH하고 점프하는 곳이 **“_dl_runtime_resolve”**라는 함수이다.

```
gdb-peda$ pdisas 0xf7fead80
Dump of assembler code from 0xf7fead80 to 0xf7feada0::	Dump of assembler code from 0xf7fead80 to 0xf7feada0:
   0xf7fead80:	push   eax
   0xf7fead81:	push   ecx
   0xf7fead82:	push   edx
   0xf7fead83:	mov    edx,DWORD PTR [esp+0x10]
   0xf7fead87:	mov    eax,DWORD PTR [esp+0xc]
   0xf7fead8b:	call   0xf7fe4f30 <_dl_fixup>
   0xf7fead90:	pop    edx
   0xf7fead91:	mov    ecx,DWORD PTR [esp]
   0xf7fead94:	mov    DWORD PTR [esp],eax
   0xf7fead97:	mov    eax,DWORD PTR [esp+0x4]
   0xf7fead9b:	ret    0xc
   0xf7fead9e:	xchg   ax,ax
End of assembler dump.
```

_dl_runtime_resolve 함수는 _dl_fixup이라는 함수를 부른다. 이 함수는 eax와 edx값을 인자로 받아오는데 다음 그림을 살펴보자.

![6](http://eliez3r.synology.me/assets/img/study/system/plt and got/6.png)

지금까지 PUSH 된 값들에 따른 스택의 상태이다. reloc_offset(ox10)을 PUSH했고 link_map 구조체(0xf7ffd940)를 PUSH 했다.

다시 한 번 정리합시다!
reloc_offsest을 push하고 Dynamic Linker를 불렀다. 그 후 link_map 구조체 포인터를 push하고 _dl_runtime_resolve를 불렀다.

push 했던 reloc_offset과 link_map 구조체 포인터를 인자로 하여 _dl_fixup함수가 불리고, _dl_fixup함수에서는 프로그램 내에서 쓰인 함수 이름의 문자열들이 저장된 STRTAB 주소와 GOT 주소 및 재배치 정보를 담고 있는 재배치 테이블인 JMPREL의 주소를 알아냈다.
STRTAB내에 있는 함수이름의 주소를 넘겨주며 _dl_lookup_symbol_x함수를 부르고, 여기서는 라이브러리 시작 주소와 라이브러리 함수 내에 있는 SYMTAB의 주소를 얻어온다.
다시 _dl_fixup함수로 돌아오면, SYMTAB 내의 실제 함수의 오프셋과 라이브러리 시작 주소를 더해 실제 함수의 주소를 알아내고 GOT에 기록한다.

![7](http://eliez3r.synology.me/assets/img/study/system/plt and got/7.png)

-----