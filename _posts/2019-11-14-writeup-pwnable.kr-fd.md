---
title: "[pwnable.kr][Toddler] fd"
tags: [pwnable.kr, Toddler's Bottle, IO]
author: eli_ez3r
key: 20191114
modify_date: 2019-11-14
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

## keyword : <u>File Discriptor</u>

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

**atoi(const char *str)  함수는 스트링 값을 정수 값으로 변환시켜 주는 함수이다.**

ex) atoi("123") -> 123    #문자열 "123"이 정수 123으로 변환

ex) atoi("a") -> 0	# 알파벳 같은 문자열은 0으로 변환된다.

16진수 `0x1234` 은 `4660` 이다.

-

```
ssize_t read(int fd, void *buf, size_t nbytes)

fd : 파일 디스크립터
void *buf :  파일을 읽어 들일 버퍼
size_t nbytes : 버퍼의 크기
return : 정상적으로 실행되었다면 읽어들인 바이트 수를 리턴, 실패시 -1을 반환
```

-

**리눅스의 File descriptor**

0 : standard input, stdin

1 : standard output, stdout

2 : standard error : stderr

> fd값을 0으로 만들어 stdin을 출력하여 키보드로 입력들 받아 "LETMEWIN"을 입력하면 되겠지?



**즉, fd값을 0으로 만들어 stdin으로 만들어 키보드로 부터 "LETMEWIN\n"을 입력받으면 flag가 출력된다.**



`````
fd@prowl:~$ ./fd 4660
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!
`````

-----