---
layout: post
title:  "[Web.kr]Level 18"
subtitle:   "[Web.kr]Level 18"
categories: Webhacking.kr(Old)
tags:
- Wargame
- webhacking.kr(Old)
- Write-up
---

![1803B656-F722-44A1-B725-4C4C33E1B6E2](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 18/1803B656-F722-44A1-B725-4C4C33E1B6E2.png)

```php
<? 
if($_GET[no]) 
{ 

if(eregi(" |/|\(|\)|\t|\||&|union|select|from|0x",$_GET[no])) exit("no hack"); 

$q=@mysql_fetch_array(mysql_query("select id from challenge18_table where id='guest' and no=$_GET[no]")); 

if($q[0]=="guest") echo ("hi guest"); 
if($q[0]=="admin") 
{ 
@solve(); 
echo ("hi admin!"); 
} 
} 

?>
```

소스코드의 중요 부위는 이부분이다.

GET방식으로 no값을 입력받아 eregi로 해당 문자열들을 필터링 하고,

mysql 쿼리문으로 id='geust'가 입력한 no값으로 쿼리를 날리고, 해당 쿼리의 첫번째 깞이 'admin'이면 문제가 풀린다.

no=1을 날리니 "hi guest"라고 출력되었다. 그러면 admin의 id를 대충 유추해보면 0 또는 2일것 같다고 유추하였다.



##### ‼️1번째 문제

그래서 no값에 0이나 2를 넣어 보냈더니 아무런 반응이 없었다.

<u>id가 guest로 설정</u>되어 있기 때문이라고 생각하고, id값과 상관없이 id를 조회하도록 하였다.

...where id = 'guest' and no=**1 or no=2** 라고 바꾸면 될 것같다.



##### ‼️2번째 문제

따라서 `1 or no=2` 라고 입력하였으나, eregi에서 <u>공백을 필터링</u>하고 있다.

따라서 공백대신 우회 할만한 것들을 찾아보니, '/'을 이용한 주석이나, '\t', '\n' 이 있다고 한다. '\t'와 '/'는 필터링 되고 있으니 '\n'을 사용하면 될 것같다.



##### ‼️3번째 문제

1\nor\nno=2 라고 입력하였더니 '\'를 urlencode시켜버려 원하는 값이 들어가지 않았다. 그래서  '\n'의 인코딩 된 값들 넣어주었다.

파이썬으로 확인해보니 '\n'은 0xa이고 urlencode형식으로 %0a라고 입력하면 될 것 같다.



정답 : `1%0aor%0ano=2`

