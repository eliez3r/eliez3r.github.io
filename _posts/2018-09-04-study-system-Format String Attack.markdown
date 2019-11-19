---
title: "포맷스트링 공격(Format String Attack)"
tags: [Foramt String Attack, printf, 포맷인자, 포맷지시자]
author: eli_ez3r
key: 20180904
modify_date: 2018-09-04
article_header:
  type: cover
  image:
    src: 
---

-----

2000년도 후반에 해커들 사이에 큰 반향을 일으키 보고서 하나가 발표되었다.
**Format String Attack**
Format String Attack이란 무엇인가? 이것은 기존에 가장 널리 사용되고 있던 Buffer Overflow 공격 기법에 견줄 만한 강력한 해킹 기법이었다. 이 해킹 기법이 발표되고 나서 그 동안 별 문제 없어 보였던 각종 프로그램들에 대한 취약점이 속속 발표되고 해당 프로그램을 제작했던 회사들은 이 취약점을 해결하기 위해 분주해지기 시작했다.

그렇다면 Format String Attack은 어떤 방식으로 이루어지는 것인가? 이것을 이해하기 위해서는 먼저 Format String이 무엇인지를 이해해야 하고, 일반 C프로그램에서 이러한 Format String이 어떻게 처리되는 지를 이해해야 한다. 

기존의 Buffer Overflow 공격 기법보다 그 난이도가 매우 높기는 하지만 이미 많은 취약점이 발견되고 exploit code가 발표되고 있다.

-----

## 0x01. Format String

 다음은 일반 C프로그램에서 흔히 찾아볼 수 있는 printf함수이다.

```c
char str[10] = "World!";
printf("Hello, %s\n", str);
```

 직관적으로 `" "` 안에 포함되어 있는 `"Hello, %s\n"`이 Format String이다.

 즉, Format String은 이를 사용하는 함수에 대해 어떤 형식 또는 **형태를 지정해 주는 문자열** 을 의미한다.



### 1. Format String 사용시 문제점

 일반적으로 프로그래머들이 printf 함수를 사용할 할때 다음과 같은 형식으로 작성한다.

```C
printf("%s", str);     //1번 코드
```

 하지만 어떤 프로그래머들은 프로그래밍을 보다 편하게 하기 위해서 다음과 같이 작성한다.

```c
printf(str);	//2번 코드
```

 프로그래밍 측면으로 봤을때 1번 코드와, 2번 코드가 잘못된 문법은 아니다. 그리고 2번 코드가 보다 짧은 코드를 사용한다. 따라서 2번 코드가 좀 더 현명해 보일 수 있다.

 **그러나 2번 소스코드를 이용하여 프로그래밍을 하는 경우는 해커들에게 프로그램의 흐름을 바꿀 수 있는 기회를 제공하게 된다.**



```c
//ex01.c
int main(){
        char str[15] = "Hello, World!\n";
        printf("%s", str);        //1번 코드
        printf(str);              //2번 코드
        return 0;
}
```

![그림1](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/1.png)

위 그림과 같이 1번, 2번 코드 모두 같은 기능을 수행하는 것을 볼 수 있다. 그러면 두 코드의 차이는 무엇일까?



### 2. 포맷 인자

 printf와 같은 Format String을 사용하는 함수는 **포맷 인자(형식 인자)**를 함수에 인자로 넘겨 특정 동작을 수행한다.

각 포맷인자는 함수의 인자로 넘겨지며, **Format String에 3개의 포맷 인자가 있으면 함수에도 3개의 인자가 있어야 한다.**

| 인자 | 입력 타입 |         출력 타입         |
| :--: | :-------: | :-----------------------: |
|  %d  |    값     |          10진수           |
|  %u  |    값     |     부호 없는 10진수      |
|  %x  |    값     |          16진수           |
|  %s  |  포인터   |          문자열           |
|  %n  |  포인터   | 지금까지 출력한 바이트 수 |



### 3. 새로운 지시자(drective) "%n"

 Format String에 사용되는 형식 지시자들 중 **`%n` 은 지금까지 출력한 바이트 수를 <u>다음 변수에 저장</u>한다.**

```c
//ex02.c
int pos, x=235, y=93;
printf("%d %n%d\n", x, &pos, y);	//%n 지시자 사용
printf("The offset was %d\n", pos);
```

![3](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/2.png)

 ex02 소스코드를 보면 **pos변수에 4라는 값이 저장**되는 것을 볼 수 있다. 이는 **"%d %n%d\n"** Format String에서 `%n` 직전까지의 **"%d "**을 봐야 한다. printf를 통해 화면에 출력될 때 "235 93"이 출력된다. **이때 %n 직전에 문자열은 "235 "이 된다. 따라서 4Byte의 문자열(공백포함)이기에 4라는 숫자가 pos변수에 저장된다.**

 

또 다른 예제를 좀더 살펴보자.

```c
//ex03.c
int pos, x=0;
char buf[20];
snprintf(buf, sizeof(buf), "%.100d%n", x, &pos);	//%n 지시자 사용
printf("position: %d\n", pos);
```

![4](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/3.png)

ex03코드 같은 경우 pos변수에 100이 저장된다. (%100d 형식지시자는 정수를 100자리로 표현)

```c
//ex04.c
#include<stdio.h>
#include<stdlib.h>
int main(){
  int A=5, B=7, count_one, count_two;
 
  //%n 포맷 스트링 예제
  //이 X포인트까지 출력한 바이트 수는 count_one에 저장되고,
  //여기의 X까지의 바이트 수는 count_two에 저장된다.
  printf("The number of bytes written up to this point X%n is being stored in count_one, and the number of bytes up to here X%n is being stored in count_two.\n",
&count_one, &count_two);//
 
 
  printf("count_one : %d\n", count_one);
  printf("count_two : %d\n", count_two);
 
  return 0;
}
```

![13](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/4.png)

![13](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/4-1.png)



------



## 0x02. printf 함수

### 1. printf 함수의 동작방식

```c
//ex05.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
int main(int argc, char **argv){
        char buf[100];
        int x;
 
        for(x=0; x<100; x++)
                buf[x]=1;
 
        if(argc != 2)
                exit(1);
 
        x=1;
        strcpy(buf, argv[1]);
        printf(buf);
        printf("\nx is %d/%#x (@ %p)\n", x, x, &x);
 
        return 0;
}
```

 ex05코드는 argv[1]로 입력받은 값과, 변수 x 정보(정수값, 16진수값, 변수주소)를 출력하는 코드이다.

![5](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/5.png)

 실행 결과를 보면 argv로 넘겨준 "Hello world"를 출력하고 x변수의 값 1, 16진수값 0x1, 그리고 변수 x의 주소 0xff905e44를 출력하였다.

![image-20190211145824684](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/6.png)

 main 함수의 스택을 살펴보면 위와 같다.

![7](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/7.png)

 실제 디버거를 통해 살펴보면 위 사진과 같다. main함수가 실행되기 이전에 먼저 인자(argument)들이 먼저 스택에 push되고 복귀주소(RET)와 베이스 포인터가 push되고나서 main함수의 지역변수를 위한 공간이 확보된다.

지역 변수가 push될 때, 배열 buf가 먼저 스택에 push되고, int형 변수 x가 push되는 것을 볼 수 있다. 이는 ex05코드상에서 buf 배열이 순차적으로 먼저 코딩되어 있기 때문이다.



![8](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/8.png)

 printf함수 역시 호출하기 전에 printf함수의 인자와 RET, EBP값이 push된다.



![image-20190211150747027](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/9.png)

 위 그림은 main함수 내부에서 printf함수를 호출할 때 스택의 모습이다.

 여기서 printf함수의 인자에 들어가는 값은 buf배열이다. (ex05코드에서 buf를 인자로 사용하므로) **이때, buf는 Format String으로 사용될 부분이고, 이는 실제 Format String이 들어가는 것이 아니라 Format String 포인터가 저장된다.(buf배열의 시작주소가 들어간다는 의미)**



![10](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/10.png)

 디버거를 통해 살펴보면 좀더 정확하게 확인 할 수 있다.

 이와 같은 스택에 대한 작업이 완료되면 printf함수는 Format String을 파싱하고 실제 출력이 이루어지게된다.

 이때, **<u>일반 문자들의 경우에는 일반 문자 그대로를 출력하고, 형식 지시자를 만나는 경우에는 해당 형식 지시자에 대한 내용을 스택에서 4Byte만큼 pop하여 출력하게 된다.(이 개념을 잊으면 안된다.🧐)</u>**  이미 앞에서 언급한 것처럼 ==**<u>이때 pop되는 것은 스택 상에서 Format String 포인터 다음에 위치한 내용</u>**==이 된다. 무슨 말인고 하면...

![image-20190211151653110](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/11.png)

 ex05 프로그램을 위와 같이 인자를 넘겨주면 printf함수는 "AAAA"를 출력하고 "%08x"를 만나면 형식지시자로 인식하고 출력을 위해 지정된 변수와는 상관없이 스택에서 4Byte만큼을 pop하여 출력하게 된다. **따라서 스택의 정보를 손쉽게 확인 할 수 있다.** 위 그림처럼 `%08x` 를 7번 입력하니 처음에 입력한 "AAAA" 문자열 값(0x41414141)이 출력되었다. **이를 통해 printf함수 인자로 부터 24Byte만큼 떨어진 곳에 buf배열이 존재한다는 것을 알 수 있다.**



![image-20190213141117013](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/12.png)

앞서 알아낸 정보를 바탕으로 스택에는 위와 같이 구성되어 있음을 알 수 있다.



### 2. Format String Attack(임의의 메모리 주소의 쓰기)

 앞서 printf함수의 특징을 살펴보았다. ex04 프로그램 예시에서 인자로 전달했던 `%x` 포맷인자 대신 `%n` 를 사용하면 어떻게 될까?

 간단히 예상해 볼 수 있는 것은 printf함수는 `%n` 포맷 인자를 만나면 스택의 내용을 pop하고 pop한 값을 주소로 이용하여 해당 주소에 지금까지 출력된 문자의 개수를 저장하게 된다. **이때 만약 pop한 주소가 RET주소가 되면 프로그램의 흐름을 바꿀 수있다.** 이것이 **Format String Attack의 주목적**이다.



```c
//ex06.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
 
int main(int argc, char* argv[]){
        char text[1024];
        static int test_val = -72;
 
        if(argc < 2){
                printf("사용법: %s <출력할 텍스트>\n", argv[0]);
                exit(0);
        }
 
        strcpy(text, argv[1]);
 
        printf("사용자 입력을 출력하기 위한 좋은 방법:\n");
        printf("%s\n", text);	// 알맞은 printf함수 사용법
 
        printf("사용자 입력을 출력위해 사용하면 안 되는 나쁜 방법:\n");
        printf(text);	// 잘못된 printf함수 사용법
 
        printf("\n");
 
        //디버깅 출력
        printf("[*]test_val @ %p = %d, %p\n", &test_val, test_val, test_val);
 
        return 0;
}
```

 ex06 프로그램을 먼저 이해해보면, text배열을 1024 Byte만큼 선언하고, test_val변수가 static int형으로 선언되었다.

![14](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/13.png)

 ex06 프로그램을 통해 `%n` 포맷인자를 이용하여 test_val(값 : -72)의 값을 바꿔보자.

 프로그램 실행 후 스택정보를 보면 printf인자와  text배열의 거리가 12Byte임을 알 수 있다.



![15](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/14.png)

> %08x 포맷인자의 크기는 8Byte이다.

 test_val 변수의 주소는 0x0804c02c이다. 따라서 buf배열의 test_val의 주소를 넣어주고 그뒤로 3번의 `%08x`  지시자를 넣고 `%n` 포맷인자를 넣으니 **test_val 변수의 값이 31로 바뀌었다.**



 **여기서 왜 test_val의 값이 바뀌었고, 왜 31이라는 값이 들어 갔는지 이해해야 된다.** 

> %n지시자는 %n지시자 앞까지의 길이를 저장한다. 해당 길이는 총 31Byte가 되고 이 값을 저장하게 되는데, 이때 %08x지시자로 3번의 pop이 일어 났다. 그 이후 스택의 최상단에는 text배열이기 때문에 해당 스택에 있는 값을 주소를 이용한다. 따라서 text배열에는 0x080498e값이 들어있고 이 주소는 test_val변수의 주소이기 때문에 test_val변수의 값에 31이 저장되게 되는 것이다.



 따라서 test_val변수의 주소 대신 RET의 주소를 넣게 되면 프로그램의 흐름을 바꿀 수 있다. 하지만 **또 다른 문제가 존재**한다. RET의 주소에 우리가 원하는 값으로 변경한다고 했을 때, 보통 특정한 코드를 수행하는 shellcode의 주소를 넣게된다.

 그 말은 즉, **주소값**을 넣어야 된다는 점이다. 주소는 32bit 환경에서 16진수 8자리로 구성되어 진다. (위에서 test_val 변수의 주소 0x0804c02c이라는 주소처럼...) 이를 해결하기 위해 다음과 같은 방법을 이용하면 된다.

|        **메모리**        | **2c** | **2d** | **2e** | **2f** |      |      |      |
| :----------------------: | :----: | :----: | :----: | :----: | :--: | :--: | :--: |
| 첫 번째 쓰기(0x0804c02c) |   aa   |   00   |   00   |   00   |      |      |      |
| 두 번째 쓰기(0x0804c02d) |        |   bb   |   00   |   00   |  00  |      |      |
| 세 번째 쓰기(0x0804c02e) |        |        |   cc   |   00   |  00  |  00  |      |
| 네 번째 쓰기(0x0804c02f) |        |        |        |   dd   |  00  |  00  |  00  |
|         **결과**         | **aa** | **bb** | **cc** | **dd** |      |      |      |

 예를들어 test_val 변수에 0xddccbbaa 값을 넣고싶다고 하면, test_val 변수의 첫 번째 바이트는 0xaa, 그 이후는 0xbb, 0xcc, 0xdd값을 넣으면 된다.

 이때 0xaa는 10진수로 170, 0xbb(187), 0xcc(204), 0xdd(221) 이다.



![18](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/15.png)

 먼저 0xaa를 넣는다고 하면 %n 포맷인자 앞으로 170Byte의 길이가 있어야 하므로 위 그림처럼 입력하면 된다.

 이제 두번째 바이트를 써야 한다. 0xbb는 10진수로 187이므로 `%x` 포맷 인자가 필요하다. **따라서 `%x` 포맷인자를 넣어야 하는데 ==<u>여기서 중요한점은 printf함수에서 포맷인자를 만나면 스택에서 4Byte만큼 pop한다는 점</u>==이다. 잊지말자. 😉** **현재 스택 맨 상단에는 text배열이므로 이부분이 %x에 들어가면 안된다. 따라서 쓸모없는 4Byte값을 중간에 넣어주어야한다.**



즉 위와 같이 1Byte씩 주소를 높여가면 된다.

![21](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/16.png)

먼저 0x0804c02c부터 1Byte씩 높여가면서 0x0804c02f까지 주소를 넣어주고, 각 주소에 넣을 값들을 `%x` 포맷인자를 이용하여 조절 해주면된다.

![22](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/17.png)

같은 방식으로 0xddccbbaa를 완성하였다. 이렇게 값들을 넣다보면 한가지 궁금증이 생기가 된다.

> 0xaabbccdd를 만든다고 할 때, 두 번째 바이트(0xcc)는 첫 번재 바이트(0xdd)보다 작다. 이럴 경우는 어떻해야 할까?



방법은 간단하다. 0xdd(221)를 만들고 나서 0xcc를 넣을 때 0x1cc(460)을 만들어서 1Byte인 0xcc만쓰여지게 하면 된다.

![24](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/18.png)

같은 방식으로 0xaabbccdd를 완성하였다. 



### 3. Format String Attack(인자에 직접 접근)

 앞서 `임의의 메모리 주소의 쓰기` 방법은 각 포맷 인자에 해당하는 값을 찾으려고 여러 메모리 주소를 건너뛰어야 했다. 그래서 Format String의 맨 앞부분에 도달할 때 까지 `%x` 포맷 인자를 사용해야만 했다. 그리고 임의의 메모리 주소에 쓰려고 3개의 추가적인 4Byte 쓰레기 값("EUNI")을 넣어야 했다.

 이제는 직접 인자에 접근하는 방법을 알아보자. 



```c
//ex07.c
#include<stdio.h>
 
int main(){
        printf("7th: %7$d, 4th: %4$05d \n", 10, 20, 30, 40, 50, 60, 70, 80);
      //%n$d는 n번째 인자를 10진수로 출력한다.
      //%7$d이므로 7번째 인자(70)를 출력한다.
        return 0;
}
```

![image-20190212230236919](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/19.png)

printf의 Foramt String을 살펴보면 `%7$d` 와 `%4$05d` 를 사용하여 총 8개의 인자 중 2개만 접근하였다. 이 방법을 이용하면 메모리에 직접 접근 할 수 있으므로 메모리 건너뛰기 위한 노력을 하지 않아도 된다. 위에서 보았던 ex06 예제를 다시 보자.

```c
//ex06.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
 
int main(int argc, char* argv[]){
        char text[1024];
        static int test_val = -72;
 
        if(argc < 2){
                printf("사용법: %s <출력할 텍스트>\n", argv[0]);
                exit(0);
        }
 
        strcpy(text, argv[1]);
 
        printf("사용자 입력을 출력하기 위한 좋은 방법:\n");
        printf("%s\n", text);
 
        printf("사용자 입력을 출력위해 사용하면 안 되는 나쁜 방법:\n");
        printf(text);
 
        printf("\n");
 
        //디버깅 출력
        printf("[*]test_val @ %p = %d, %p\n", &test_val, test_val, test_val);
 
        return 0;
}
```

![29](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/20.png)

 ex06 예제에서 Format String의 맨 앞부분은 4번째 포맷 인자에 해당한다. 이곳에 접근하려고 `%x` 포맷 인자 4개를 사용해 메모리를 건너뛰는 **대신 달러 기호(`$`)**를 사용해 직접 접근하는 방법을 사용할 수 있다.

> **달러 기호(`$`)는 특수 문자이기 때문에 커맨드라인에서 사용하려면 앞에 역슬래시(`\`)문자를 붙여야 한다.** 이렇게 하면 명령 셀이 달러 기호(`$`)를 특수 문자로 인식하지 않는다.

 직접적인 인자 접근법은 메모리 주소에 데이터를 쓰는 과정도 단순화시킨다. 이 방법을 사용하면 메모리에 바로 접근할 수 있으므로 출력 바이트 카운트를 증가시키기 위한 4Byte 쓰레기값을 입력하지 않아도 된다.

![image-20190212231352003](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/21.png)

보단 간결하게 메모리에 쓰기에 가능해졌다.



### 4. Format String Attack(쇼트 쓰기 기법)

 Format String Attack을 단순화 하는 또 다른 방법은 **쇼트 쓰기 기법** 이다.

쇼트(short)는 보통 2Byte 워드이다. 그리고 포맷 인자는 쇼트를 다루는 특별한 방법을 가진다. 쇼트 쓰기는 2Byte 쇼트를 쓰는 포맷 스트링 공격으로 사용할 수 있다.

![22](http://eliez3r.synology.me/assets/img/study/system/Format String Attack/22.png)

앞서 1Byte씩 0xaa, 0xbb, 0xcc, 0xdd를 메모리에 썼다면, 쇼트 쓰기 기법은 0xbbaa, 0xddcc 2Byte씩 메모리에 쓰는 기법이다.



## 0x03. 정리

 임의의 메모리 주소에 데이터를 쓸 수 있다는 것은 프로그램 실행 흐름을 제어할 수 있음을 의미한다. 가장 대표적으로 Stack BufferOverflow에서 사용했던 방법인 Stack Frame의 리턴 주소(RET)를 덮어쓰는 방법이 있다. Foramt String Attack은 임의의 메모리 주소에 데이터를 쓸 수 있으므로 다양한 공격에 활용 할 수 있다.



