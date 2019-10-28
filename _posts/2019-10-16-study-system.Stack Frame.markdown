---
title: "스탭프레임(Stack Frame) 이란?"
tags: [stack, frame, prolog, epilog]
author: eli_ez3r
key: 20191016
modify_date: 2019-10-16
article_header:
  type: cover
  image:
    src: http://eliez3r.synology.me/assets/img/study/system-logo.png
---

스택 프레임(Stack Frame)이란 함수가 호출될 때, 그 함수만의 스택 영역을 구분하기 위하여 생성되는 공간이다. 이 공간에는 함수와 관계되는 지역 번수, 매개변수가 저장되며, 함수 호출 시 할당되며, 함수가 종료되면서 소멸한다.

이 영역을 표현하기 위해 **함수 프롤로그(Prolog)**와 **함수 에필로그(Epilog)**라는 것을 수행한다.



### 1) 함수 프롤로그(Prolog)

함수 프롤로그란 함수가 호출되면 그 함수의 영역을 설정하기 위해 2개의 명령어를 사용한다.

`push ebp`, `mov ebp, esp` 

<div class="item">
  <div class="item__image">
    <img class="image image--lg" src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571364796122.png"/>
  </div>
  <div class="item__content">
    <div class="item__header">
      <h4>함수 프롤로그</h4>
    </div>
    <div class="item__description">
      <p>ebp를 push 함으로써 caller 함수(자신을 호출 한 함수)의 ebp를 스택에 저장한다. 그리고 esp를 ebp에 복사함으로써 esp를 ebp주소로 설정한다. ebp는 함수의 기준점을 나타내므로, ebp를 기준으로 Return Address나 argc, argv 그리고 함수의 지역변수 등의 위치를 쉽게 알 수 있다.</p>
    </div>
  </div>
</div>

-----

그러면 실제 스택 프레임이 어떻게 생겨나는지 어셈블리 코드와 스택을 직접 살펴보면서 알아보자.

확인을 위한 코드는 다음과 같다.

```c
//stack.c
//gcc -m32 -mpreferred-stack-boundary=2 -fno-stack-protector -z execstack -no-pie -fno-pic -o stack stack.c
void func(){
        char str1[10];
        printf("Called func()\n");
}

int main(){
        char str2[20];
        printf("Called main()\n");
        func();

        return 0;
}
```

main함수 내에서 func함수를 호출하는 단순한 코드이다.

```
[gate@localhost /tmp]$ gdb -q stack
(gdb) set dis int
(gdb) disas main
Dump of assembler code for function main:
0x80483e8 <main>:       push   ebp
0x80483e9 <main+1>:     mov    ebp,esp
0x80483eb <main+3>:     sub    esp,20
0x80483ee <main+6>:     push   0x804846f
0x80483f3 <main+11>:    call   0x8048308 <printf>
0x80483f8 <main+16>:    add    esp,4
0x80483fb <main+19>:    call   0x80483d0 <func>
0x8048400 <main+24>:    xor    eax,eax
0x8048402 <main+26>:    jmp    0x8048404 <main+28>
0x8048404 <main+28>:    leave
0x8048405 <main+29>:    ret
End of assembler dump.
```

main함수를 살펴보면 앞서말한 함수 프롤로그와 에필로그가 보이고 중간에 printf함수와 func함수를 호출하기 위한 코드들이 보여진다.

main함수 처음에 브레이크포인트를 걸고 하나씩 살펴보자.

```
Breakpoint 1, 0x80483e8 in main ()
4: /x $eip = 0x80483e8
3: /x $ebp = 0xbffffd18
2: /x $esp = 0xbffffcfc
1: x/11i {<text variable, no debug info>} 0x80483e8 <main>
0x80483e8 <main>:       push   ebp
0x80483e9 <main+1>:     mov    ebp,esp
0x80483eb <main+3>:     sub    esp,20
0x80483ee <main+6>:     push   0x804846f
0x80483f3 <main+11>:    call   0x8048308 <printf>
0x80483f8 <main+16>:    add    esp,4
0x80483fb <main+19>:    call   0x80483d0 <func>
0x8048400 <main+24>:    xor    eax,eax
0x8048402 <main+26>:    jmp    0x8048404 <main+28>
0x8048404 <main+28>:    leave
0x8048405 <main+29>:    ret
```

main함수에 브레이크포인트를 걸고 `push ebp` 명령어를 수행하기 직전에 상태는 위와 같다.

ebp의 값은 `0xbffffd18`의 값을 가지고있다. 이 값은 앞서 말한것 처럼 main함수를 호출하는 caller의 ebp값이다. main함수가 끝나고 나서 caller의 ebp값으로 복원하기 위해 스택에 저장한다.

현재 esp의 값은 0xbffffcfc이고 스택의 상태를 살펴보면 다음과 같다.

```sh
(gdb) x/8x $esp
0xbffffcfc:     0x400309cb      0x00000001      0xbffffd44      0xbffffd4c
0xbffffd0c:     0x40013868      0x00000001      0x08048320      0x00000000
```

`push ebp` 명령어가 실행되면 스택에 ebp의 값이 저장되고 esp값은 4가 감소 될 것이다.

```sh
(gdb) x/8x $esp
0xbffffcf8:     0xbffffd18      0x400309cb      0x00000001      0xbffffd44
0xbffffd08:     0xbffffd4c      0x40013868      0x00000001      0x08048320
```

`push ebp` 명령어가 수행되고 나서 스택이다. main함수를 호출한 caller의 ebp값(`0xbffffd18`)이 스택에 추가된 것을 알 수 있다.

그리고 나서 현재 main함수의 스택 프레임을 설정하기 위해 `mov esp, ebp` 명령어를 수행한다. 스택을 그림으로 표현하면 다음과 같다.

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571371679806.png" width="600px">

이렇게 함수 프롤로그를 통하여 main함수의 스택 프레임이 생성된다. 

그리고 나서 본격적으로 main함수의 명령들이 실행된다.

```c
//stack.c
void func(){
        char str1[10];
        printf("Called func()\n");
}

int main(){
        char str2[20];
        printf("Called main()\n");
        func();

        return 0;
}
```

```
0x80483e8 <main>:       push   ebp
0x80483e9 <main+1>:     mov    ebp,esp
0x80483eb <main+3>:     sub    esp,20
0x80483ee <main+6>:     push   0x804846f
0x80483f3 <main+11>:    call   0x8048308 <printf>
0x80483f8 <main+16>:    add    esp,4
```

main함수에서 str2배열을 20byte 선언을 하였다. 따라서 해당 배열의 공간을 할당하기 위해 `sub esp, 20` 명령어가 들어가 있음을 확인 할 수있다.

그리고나서 printf함수를 호출하기 위해 `push 0x804846f` 명령어로 스택에 특정 주소를 push한다. 해당 주소에는 printf에서 출력될 문자열("Called main()\n")이 들어있을 것이다.

```
(gdb) x/s 0x804846f
0x804846f <_IO_stdin_used+19>:   "Called main()\n"
```

printf함수가 호출되고 나면 `add esp, 4` 명령어를 통해 printf함수를 호출하기 위해 push했던 영역 4byte를 해제한다.

현재까지 스택의 상태를 살펴보면 다음과 같다.

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571373529487.png" width="600px">

이제 func 함수를 호출한다. 

```
Dump of assembler code for function func:
0x80483d0 <func>:       push   ebp
0x80483d1 <func+1>:     mov    ebp,esp
0x80483d3 <func+3>:     sub    esp,12
0x80483d6 <func+6>:     push   0x8048460
0x80483db <func+11>:    call   0x8048308 <printf>
0x80483e0 <func+16>:    add    esp,4
0x80483e3 <func+19>:    leave
0x80483e4 <func+20>:    ret
```

여기서 caller는 main함수가 되고 callee는 func함수가 된다.

func함수도 마찬가지로 함수 프롤로그를 통해 ebp를 저장한다. 이때 ebp는 caller의 ebp, 즉 main함수의 ebp값이 저장된다. (func의 RET값은 main함수에서 func를 호출하고 다음 주소가 들어가게 된다.)

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571373928369.png" width="600px">

그리고 str1배열의 10byte 공간을 할당하는데 32bit체제에서는 4byte(32bit) 배수로 공간을 할당하므로 12byte의 공간을 할당하는 것을 볼 수 있다.

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571374299559.png" width="600px">

-----

### 2) 함수 에필로그(Epilog)

함수 에필로그는 callee 함수가  끝나서 다시 caller 함수로 돌아갈 때 스택을 정리하는 과정이다. 다시말해, 스택을 다시 callee함수를 호출하기 전 상태로 되돌리는 과정을 말한다.

`leave`, `ret` 명령어 2개를 사용한다.

여기서 `leave` 명령어는 실제 2개의 명령어로 이루어져 있다. 바로 `mov esp, ebp`, `pop ebp` 이다.

그리고 `ret`명령어도 역시 2개의 명령어로 이루어져 있는데, `pop eip`, `jmp eip` 이다.

<div class="item">
  <div class="item__image">
    <img class="image image--lg" src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571364796123.png"/>
  </div>
  <div class="item__content">
    <div class="item__header">
      <h4>함수 에필로그</h4>
    </div>
    <div class="item__description">
      <p>leave 명령어로 인해 esp가 현재 함수의 ebp를 가리키게 되고, pop ebp를 통해 sfp에 저장되었던 caller의 ebp값으로 ebp가 설정된다.</p>
        <p>이어서 ret 명령어로 인해 pop eip 명령이 수행되고, RET값이 eip로 설정된다. 그리고 jmp eip를 통해 설정된 RET주소로 점프하게 된다. 여기서 RET값은 callee를 호출한 명령어의 다음주소를 담고있다.
        </p>
    </div>
  </div>
</div>

-----

func함수가 종료될때 에필로그를 자세히 살펴보자.

```
Dump of assembler code for function func:
0x80483d0 <func>:       push   ebp
0x80483d1 <func+1>:     mov    ebp,esp
0x80483d3 <func+3>:     sub    esp,12
0x80483d6 <func+6>:     push   0x8048460
0x80483db <func+11>:    call   0x8048308 <printf>
0x80483e0 <func+16>:    add    esp,4
0x80483e3 <func+19>:    leave
0x80483e4 <func+20>:    ret
```

앞서 `leave` 명령어는 `mov esp, ebp`, `pop ebp`로 나누어 진다고 하였다.

그리고 `ret`명령어는 `pop eip`, `jmp eip` 로 나누어 진다.

현재 스택의 모습은 다음과 같다.

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571374299559.png" width="600px">

이상태에서 `mov esp, ebp`명령이 수행되면

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571374563968.png" width="600px">

esp값이 ebp를 가리키게 된다. 그리고 `pop ebp` 명령을 수행하면, 현재 가리키고 있는 esp의 값을 ebp로 넣고 esp값이 4증가한다. 그러면 ebp값은 caller(main함수)의 ebp 값이 들어간다.

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571374657748.png" width="600px">

이제 `ret` 명령에 의해 `pop eip`, `jmp eip`를 수행하게 된다.

`pop eip` 명령에 의해 esp가 가리키고 있는 값이 eip로 들어가고 esp는 4증가 한다.

<img src="http://eliez3r.synology.me/assets/img/study/system/Stack Frame/1571374760558.png" width="600px">

그리고 eip의 값으로 점프하게 되는데 해당 eip주소 (0x08048400)의 값은 main함수에서 func함수를 호출한 다음의 주소가 된다.

```
Dump of assembler code for function main:
0x80483e8 <main>:       push   ebp
0x80483e9 <main+1>:     mov    ebp,esp
0x80483eb <main+3>:     sub    esp,20
0x80483ee <main+6>:     push   0x804846f
0x80483f3 <main+11>:    call   0x8048308 <printf>
0x80483f8 <main+16>:    add    esp,4
0x80483fb <main+19>:    call   0x80483d0 <func>
0x8048400 <main+24>:    xor    eax,eax
0x8048402 <main+26>:    jmp    0x8048404 <main+28>
0x8048404 <main+28>:    leave
0x8048405 <main+29>:    ret
End of assembler dump.
```

-----

이와 같이 main함수에서 func함수를 호출하고, func함수가 종료되면서 스택 프레임이 어떻게 생성되고 소멸되는지 살펴보았다.

이러한 스택 프레임 과정을 통해서 RET값이나 SFP값을 조작하여 수많은 공격 기법들이 존재 한다.

따라서 해당 공격 기법들을 이해하기 위해서는 스택프레임에 대하여 정확하게 알고 있어야 한다.

-----

