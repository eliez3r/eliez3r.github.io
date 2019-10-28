---
title: "[Web.kr]Level 11"
tags: [wargame, webhacking.kr(old), writeup]
author: eli_ez3r
key: 20180903
modify_date: 2018-09-03
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 11/image-20180730141047950.png" width="400px">



$pat에 이상한 정규 표현식들이 들어있고, if문을 살펴보면 pat와 val값이 같으면 풀리는 것 같다.



먼저 php 정규포현식에 대해 알아보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 11/image-20180730141934532.png" width="500px">

대괄호 안에 있는 패턴의 일부를 "캐릭터 클래스"라고 하는데, '캐릭터 클래스'에서 사용할 수 있는 메타 문자는 다음과 같다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 11/image-20180730142054910.png" width="400px">



다음은 이스퀘이프 시퀀스에 대한 정규식이다,

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 11/image-20180730142148309.png" width="400px">



위 내용들을 종합하여 문제의 정규식을 해석하면, [참고](http://www.nextree.co.kr/p4327/)

`"/[1-3][a-f]{5}_.\*223.52.120.172.*\tp\ta\ts\ts/" `

/ : 구분기호

[1-3] : 1부터 3까지 중 하나

[a-f]{5}_ : a부터 f까지 중 하나의 문자를 5번 반복하는 문자를 찾음, 마지막에 '_'를 붙임 ( x{n} : 'x'를 n 번 반복 )

\t : tab을 의미함. (url endcode : %09)



> GET	방식에서 val 파라미터에 값을 전달하므로 tab을 url encoding 하여 전달해야 한다.



따라서 조건대로 val 값에 넣어주면 된다.

`1abcde_[접속 아이피]%09ap%09a%09s%09s`

맨 앞 숫자는 1-3중 아무거나, 그뒤 문자도 a-f중 아무거나 5개 넣으면 된다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 11/image-20180730193453562.png" width="500px">



