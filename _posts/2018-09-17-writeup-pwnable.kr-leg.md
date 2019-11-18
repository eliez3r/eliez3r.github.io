---
title: "[pwnable.kr]Toddler/leg"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180917
modify_date: 2018-09-17
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

```
Daddy told me I should study arm.
But I prefer to study my leg!

Download : http://pwnable.kr/bin/leg.c
Download : http://pwnable.kr/bin/leg.asm

ssh leg@pwnable.kr -p2222 (pw:guest)
```

문제에서 `leg.c` 파일과 `leg.asm` 파일이 주어진다.

-----

### 0x00. Analysis

```c
#include <stdio.h>
#include <fcntl.h>
int key1(){
        asm("mov r3, pc\n");
}
int key2(){
        asm(
        "push   {r6}\n"
        "add    r6, pc, $1\n"
        "bx     r6\n"
        ".code   16\n"
        "mov    r3, pc\n"
        "add    r3, $0x4\n"
        "push   {r3}\n"
        "pop    {pc}\n"
        ".code  32\n"
        "pop    {r6}\n"
        );
}
int key3(){
        asm("mov r3, lr\n");
}
int main(){
        int key=0;
        printf("Daddy has very strong arm! : ");
        scanf("%d", &key);
        if( (key1()+key2()+key3()) == key ){
                printf("Congratz!\n");
                int fd = open("flag", O_RDONLY);
                char buf[100];
                int r = read(fd, buf, 100);
                write(0, buf, r);
        }
        else{
                printf("I have strong leg :P\n");
        }
        return 0;
}
```

`leg.c` 파일의 소스 코드는 위와 같다. main함수를 살펴보면 key1(), key2(), key3()의 리턴 값을 다 더한 값과, 사용자가 입력한 값이 같으면 문제가 풀린다.

따라서 key1~3 함수의 리턴 값을 각각 구해야 한다.

----

먼저 `leg.asm`에서 main함수 부분을 살펴보자. (key1~3 호출 부분)

```
----(생략)----
0x00008d64 <+40>:    bl      0xfbd8 <__isoc99_scanf>
0x00008d68 <+44>:    bl      0x8cd4 <key1>
0x00008d6c <+48>:    mov     r4, r0
0x00008d70 <+52>:    bl      0x8cf0 <key2>
0x00008d74 <+56>:    mov     r3, r0
0x00008d78 <+60>:    add     r4, r4, r3
0x00008d7c <+64>:    bl      0x8d20 <key3>
0x00008d80 <+68>:    mov     r3, r0
0x00008d84 <+72>:    add     r2, r4, r3
0x00008d88 <+76>:    ldr     r3, [r11, #-16]
0x00008d8c <+80>:    cmp     r2, r3
----(생략)----
```

asm에서 함수의 리턴 값은 `r0`로 들어간다. key1함수의 리턴 값은 `r4`로 들어가고. key2함수의 리턴 값은 `r3`로 들어간 후 `r3`와 `r4`가 더해서 `r4`로 들어간다. 이후 key3의 리턴 값은 `r3`로 들어가고 다시 `r4`와  `r3`와 더해서 `r2`로 들어간다.

이후 사용자가 입력한 값 r3와 key1~3함수의 리턴 값을 더한  `r2`와 비교를 하게 된다.



-----

`leg.asm`에서 key1함수의 부분이다.

```
0x00008cd4 <+0>:     push    {r11}           ; (str r11, [sp, #-4]!)
0x00008cd8 <+4>:     add     r11, sp, #0
0x00008cdc <+8>:     mov     r3, pc
0x00008ce0 <+12>:    mov     r0, r3
0x00008ce4 <+16>:    sub     sp, r11, #0
0x00008ce8 <+20>:    pop     {r11}           ; (ldr r11, [sp], #4)
0x00008cec <+24>:    bx      lr
```

함수의 리턴 값은 `r0`에 들어가기 때문에 `r0` 레지스터를 중심으로 추적해보면 된다.

`key1+12`부분에서 `r0`에 `r3`값이 들어간다. `r3`를 추적하면 `r3`에는 `pc`값이 들어간다.

`pc`는 실행되는 명령어의 주소를 가리킨다. 때문에 `0x8cdc`를 가리킬 것 같지만 ARM에서는 다르다.([ARM의 Pipeline]( https://eliez3r.github.io/post/2019/11/14/study-arm-pipeline.html ) 를 참조하자.)

<img src="http://eliez3r.synology.me/assets/img/writeup/pwnable.kr/leg/1.png" width="600px">



ARM Pipeline구조에 의해서 실제 실행되는 주소는 `0x8ce4`가 된다. 따라서 key1함수의 리턴값은 `0x8ce4`이다.

-----

이제 key2함수 부분을 살펴보자.

```
 0x00008cf0 <+0>:     push    {r11}           ; (str r11, [sp, #-4]!)
 0x00008cf4 <+4>:     add     r11, sp, #0
 0x00008cf8 <+8>:     push    {r6}            ; (str r6, [sp, #-4]!)
 0x00008cfc <+12>:    add     r6, pc, #1
 0x00008d00 <+16>:    bx      r6
 0x00008d04 <+20>:    mov     r3, pc
 0x00008d06 <+22>:    adds    r3, #4
 0x00008d08 <+24>:    push    {r3}
 0x00008d0a <+26>:    pop     {pc}
 0x00008d0c <+28>:    pop     {r6}            ; (ldr r6, [sp], #4)
 0x00008d10 <+32>:    mov     r0, r3
 0x00008d14 <+36>:    sub     sp, r11, #0
 0x00008d18 <+40>:    pop     {r11}           ; (ldr r11, [sp], #4)
 0x00008d1c <+44>:    bx      lr
```

똑같은 방법으로 `r0`부터 추적을 시작해보면, `r0`에는 `r3`가 들어가고, `r3`에는 `pc`가 들어간 후 `4`가 더해진다. 여기서도 똑같이 ARM Pipeline 구조에 의해서 pc값에는 `0x8d04`가 아닌 `0x8d08`의 값이 들어가고, 여기서 4를 더한 `0x8d0c`값이 key2함수의 리턴 값이 된다.

-----

마지막으로 key3함수를 살펴보자.

```
0x00008d20 <+0>:     push    {r11}           ; (str r11, [sp, #-4]!)
0x00008d24 <+4>:     add     r11, sp, #0
0x00008d28 <+8>:     mov     r3, lr
0x00008d2c <+12>:    mov     r0, r3
0x00008d30 <+16>:    sub     sp, r11, #0
0x00008d34 <+20>:    pop     {r11}           ; (ldr r11, [sp], #4)
0x00008d38 <+24>:    bx      lr
```

마찬가지로 `r0`에는 `r3`가 들어가고 `r3`에는 `lr`값이 들어간다.

`lr` 레지스터는 해당 함수가 끝나고 돌아갈 주소값을 가지고 있다. (Intel Core에서 SFP값과 같다.)

```
 0x00008d68 <+44>:    bl      0x8cd4 <key1>
 0x00008d6c <+48>:    mov     r4, r0
 0x00008d70 <+52>:    bl      0x8cf0 <key2>
 0x00008d74 <+56>:    mov     r3, r0
 0x00008d78 <+60>:    add     r4, r4, r3
 0x00008d7c <+64>:    bl      0x8d20 <key3>
 0x00008d80 <+68>:    mov     r3, r0
 0x00008d84 <+72>:    add     r2, r4, r3
 0x00008d88 <+76>:    ldr     r3, [r11, #-16]
 0x00008d8c <+80>:    cmp     r2, r3
```

main함수에서 key3 함수가 실행되고 복귀 할 주소는 `0x8d80`이므로, key3의 리턴 값은 `0x8d80`이다.

-----

이제 key1함수 부터 key3함수까지의 리턴 값을 더하면 된다.

- key1의 리턴 값 : `0x8ce4`
- key2의 리턴 값 : `0x8d0c`
- key3의 리턴 값 : `0x8d80`

```
>>> 0x8ce4 + 0x8d0c + 0x8d80
108400
```

-----

### 0x01. Exploit

```
/ $ ./leg
Daddy has very strong arm! : 108400
Congratz!
My daddy has a lot of ARMv5te muscle!
```

끄읏..!

-----

