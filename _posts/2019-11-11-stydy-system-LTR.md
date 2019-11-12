---
title: "Return To Library(RTL) 넌 뭐냐?"
tags: [RTL, Return To Library, plt, got]
author: eli_ez3r
key: 20191111
modify_date: 2019-11-11
article_header:
  type: cover
  image:
    src: 
---

### 0x00. 개요

RTL(Return To Library)는 공유 라이브러리에 있는 함수의 주소를 이용해서 바이너리에 존재하지 않는 함수를 사용할 수 있다.

> 예를들어, 사용자가 입력한 소스코드 내에는 printf함수와 scanf함수만 정의하고 사용하더라도, 라이브러리 내에 있는 system함수를 사용할 수 있다.



**DEP(Data Execution Prevention) 메모리 보호기법[^1],   또는 NX bit(Never Execute Bit)[^2]**가 적용된 스택을 우회하기 위해 사용한다.

> 윈도우에서는 DEP라고 하고 리눅스에서는 NX Bit라고 표현한다.
>
> 둘다 같은 보호기법이다.





-----

[^1]:테이터 실행 방지로 스택이나 힙에서 쉘 코드 실행을 막아주는 메모리 보호기법이다.
[^2]: NX 특성으로 지정된 모든 메모리 구역은 데이터 저장을 위해서만 사용되며, 프로세서 명령어가 그 곳에 상주하지 않음으로써 실행되지 않도록 만들어 준다.

