---
title: "[pwnable.kr]Toddler/random"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180916
modify_date: 2018-09-16
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---



```c
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

        unsigned int key=0;
        scanf("%d", &key);

        if( (key ^ random) == 0xdeadbeef ){
                printf("Good!\n");
                system("/bin/cat flag");
                return 0;
        }

        printf("Wrong, maybe you should try 2^32 cases.\n");
        return 0;
}
```

-----

### 0x00. Analysis

이 문제는 rand() 함수의 특징과, xor의 특징을 알고 있는지 묻는 문제인것 같다.

rand() 함수는 우리가 생각하는 랜덤과는 다르다. ~~컴퓨터는 멍청하기 때문에~~

컴퓨터는 똑같은 패턴으로 랜덤 값을 생성한다.

쉽게 이해하기 위해 다음과 같은 코드를 작성하고 실행해 보았다.

```c
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

        printf("%d\n", random);
        return 0;
}
```

문제와 똑같이 rand()함수를 실행하여 random 값을 생성하고 그대로 출력하였다.

```
root@ubuntu:~/pwnable/random# ./test
1804289383
root@ubuntu:~/pwnable/random# ./test
1804289383
root@ubuntu:~/pwnable/random# ./test
1804289383
root@ubuntu:~/pwnable/random# ./test
1804289383
root@ubuntu:~/pwnable/random# ./test
1804289383
```

실행해보면 매번 똑같은 값을 생성한다.  이처럼 컴퓨터는 미리 정해진 패턴으로 랜덤값을 생성한다. 따라서 우리는 랜덤값을 손쉽게 구할 수 있다. (이미 위에서 실행으로 구했음....;;)

-----

이제 xor의 특징을 알아야 한다.

```c
if( (key ^ random) == 0xdeadbeef ){
                printf("Good!\n");
                system("/bin/cat flag");
                return 0;
}
```

if문을 통해 `key` 값과 `random` 값을 xor한 값이 `0xdeadbeef`이여야 flag를 획득할 수 있다.

xor의 특징은 `A` xor `B` = `C` 라고 할 때, `A` xor `C` = `B` 이다.

따라서 우리는 `random` 값과 `0xdeadbeef` 값을 알고 있기 때문에 `key`값을 구할 수 있다.

위에서 random값은 `1804289383`인것을 구했다.

```python
>>> 1804289383 ^ 0xdeadbeef
3039230856
```

`key` 값은 `3039230856`이다.

-----

문제는 key값을 넣으면 풀리는 문제이기 때문에 이제 넣기만 하면 된다. (쉽죠잉~~??)

-----

### 0x01. Exploit

```
random@prowl:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
```

##### python

```python
from pwn import *

conn = ssh("random", "pwnable.kr", port=2222, password="guest")
p = conn.process("./random")

random = 1804289383
key = random ^ 0xdeadbeef

p.sendline(str(key))
print p.recv()

p.close()
conn.close()
```

```
root@ubuntu:~/pwnable/random# python ex.py
[+] Connecting to pwnable.kr on port 2222: Done
[*] fd@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.4.179
    ASLR:     Enabled
[+] Starting remote process './random' on pwnable.kr: pid 124967
Good!
Mommy, I thought libc random is unpredictable...

[*] Stopped remote process 'random' on pwnable.kr (pid 124967)
[*] Closed connection to 'pwnable.kr'
```

-----