---
title: "[Web.kr]Level 23"
tags: [wargame, webhacking.kr(old), writeup]
author: eli_ez3r
key: 20180903
modify_date: 2018-09-03
article_header:
  type: cover
  image:
    src: 
---

![4](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 23/image-20180726154434963.png)

`<sciprt>alert(1);</script>` 를 넣는 것이 미션이란다. 



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 23/image-20180726154848452.png" width="300px">

그래서 넣어보았다. "no hack"이라고 출력된다. 앞서 푼 문제들로 추정해 봤을 때, 필터링이 있는 것 같았다. 그래서 어떤 문자가 필터링 되는지 여러가지 문자들을 넣어 보았다.

'<', '>', '(', ')' 등 특수문자들과 숫자는 모두 가능하지만, 알파벳이 연속 2자리 이상 오면 무조건 필터링 된다.

php나 c같은 백엔드 언어의 경우 문자열에서 %00(NULL)을 만나면 문자열의 끝이라고 생각하고 더 이상 필터링 하지 않는다.



`%00<script>alert(1);</script>` 를 넣으니 Clear!

