---
title: "[pwnable.kr]Toddler/collision"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20180051
category: write-up
date: 2018-09-12 00:00:00 +0900
modify_date: 2018-09-12
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png

---

-----

```
Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

ssh col@pwnable.kr -p2222 (pw:guest)
```

-----

```c
#include <stdio.h>
#include <string.h>

unsigned long hashcode = 0x21DD09EC;

unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

### 0x01. Analysis

입력한 값의 길이는 20자리여야 하며, 입력한 값을 4바이트씩(int*) 5번 더한다.

그 값이 `hashcode(0x21DD09EC)`와 같으면 문제가 풀린다.



따라서 hashcode를 5로 나누면 `0x6c5cec8` 이라는 값이 나오는데, 실제 이 값에 5를 곱하면 `0x21dd09e8`라는 값이 나온다. (나눌 때 버림이 되어서...)

hashcode와 차이가 4이므로 

따라서 4번은 `0x6c5cec9`로 넣고 마지막에는 1작은 `0x6c5cec8`을 넣으면 된다.

-----

### 0x02. Exploit

```
col@prowl:~$ ./col `python -c 'print "\xc9\xce\xc5\x06"*4+"\xc8\xce\xc5\x06"'`
daddy! I just managed to create a hash collision :)
```

-----

