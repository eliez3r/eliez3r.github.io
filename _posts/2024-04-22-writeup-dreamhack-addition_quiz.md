---
title: "[DreamHack] addition-quiz Write-up"
tags: [dreamhack, pwntools, writeup, misc, beginner]
author: eli_ez3r
key: 20240422
modify_date: 2024-04-22
article_header:
  type: cover
  image:
    src: /assets/img/Dreamhack_logo.svg
---

## Description

```
랜덤한 2개의 숫자를 더한 결과가 입력 값과 일치하는지 확인하는 과정을 50번 반복하는 프로그램입니다. 모두 일치하면 flag 파일에 있는 플래그를 출력합니다. 알맞은 값을 입력하여 플래그를 획득하세요.
```
-----

## Code (chall.c)

```c
// Name: chall.c
// Compile Option: gcc chall.c -o chall -fno-stack-protector

#include <stdio.h>
#include <signal.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <time.h>

#define FLAG_SIZE 0x45

void alarm_handler() {
    puts("TIME OUT");
    exit(1);
}

void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    signal(SIGALRM, alarm_handler);
}

int main(void) {
    int fd;
    char *flag;

    initialize();
    srand(time(NULL)); 

    flag = (char *)malloc(FLAG_SIZE);
    fd = open("./flag", O_RDONLY);
    read(fd, flag, FLAG_SIZE);
    close(fd);

    int num1 = 0;
    int num2 = 0;
    int inpt = 0; 

    for (int i = 0; i < 50; i++){
        alarm(1);
        num1 = rand() % 10000;
        num2 = rand() % 10000;
        printf("%d+%d=?\n", num1, num2);
        scanf("%d", &inpt);

        if(inpt != num1 + num2){
            printf("Wrong...\n");
            return 0;
        }
    } 
    
    puts("Nice!");
    puts(flag);

    return 0;
}
```
---

## Analysis

문제 설명에 쓰여져 있는대로 랜덤 값을 생성하여 일치하는지 확인하는 프로그램이다.

프로그램을 실행해보았다.

```shell
$ ./chall 
5877+1109=?
TIME OUT
```

프로그램 실행 시 랜덤 값 2개를 더하고 답을 입력 받는데,
문제는 1초 정도 안에 답을 입력해야 한다.
내가 암산이 아무리 빠르다고 해도 내 손가락이 따라가지 못할것 같다.

즉, 컴퓨터를 이용해서 문제를 풀어야할것 같다.

---

## Solve

해당 문제 힌트에도 나와있지만, `Pwntools`을 이용해서 문제를 풀어보려고 한다.

> [PwnTools 정리](https://eliez3r.github.io/post/2019/11/15/study-manual-pwntools.html)


작성한 코드는 아래와 같다.

```python
from pwn import *

with remote("host3.dreamhack.games", 14055) as conn:
    for _ in range(0, 51):
        res = conn.recv().decode('utf-8')
        print(f"[>] {res}")

        if '=' in res:
            a = res[:res.index('=')]
            num1, num2 = a.split("+")
            result = int(num1) + int(num2)

            conn.sendline(str(result))
```

코드는 정말 간단하다. 문자열 값으로 받아온 문제를 파싱하여 정수형인 `num1` 과 `num2`를 구하고 그 값을 합친 값을 다시 문자열로 입력해주는 코드이다.

```shell
.... 생략 ....
[>] 3271+3075=?
[>] 282+2237=?
[>] 7062+3892=?
[>] Nice!
DH{ masking }
```

결과적으로 자동으로 계산하여 flag 값 까지 구할 수 있었다.
프로그램을 분석해서 취약점을 발견하는 문제라기 보다는
pwntools를 이용하는 방법을 공부하기 위한 문제이다.

-----

