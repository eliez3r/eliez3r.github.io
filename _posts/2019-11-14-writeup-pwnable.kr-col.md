---
title: "[pwnable.kr][Toddler] col"
tags: [pwnable.kr, Toddler's Bottle, writeup]
author: eli_ez3r
key: 20191114
modify_date: 2019-11-14
article_header:
  type: cover
  image:
    src: /assets/img/write-up/pwnable_kr.png
---

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

입력한 값의 길이는 20자리여야 하며, 입력한 값을 4바이트씩(int*) 5번 더한다.

그 값이 hashcode와 같으면 문제가 풀린다.



따라서 hashcode를 5로 나누면 0x6c5cec8 이라는 값이 나오는데, 실제 이 값에 5를 곱하면 0x21dd09e8라는 값이 나온다. (반올림 되어서...)

그래서 0x6c5cec9로 1을 더 증가시키고 5를 곱하면 0x21dd09ed라는 hashcode보다 1더 큰 수가 나온다. 

따라서 4번은 10x6c5cec91로 넣고 마지막에는 1작은 10x6c5cec81을 넣으면 된다.

```
col@prowl:~$ ./col `python -c 'print "\xc9\xce\xc5\x06"*4+"\xc8\xce\xc5\x06"'`
daddy! I just managed to create a hash collision :)
```

-----