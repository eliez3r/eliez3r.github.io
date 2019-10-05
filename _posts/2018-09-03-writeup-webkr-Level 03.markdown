---
title: "[Web.kr]Level 03"
tags: [Wargame, webhacking.kr(Old), Write-up]
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729124055653.png" width="300px">

네모로직 문제가 나왔다. 네모칸을 클릭하면 검정색으로 변하게 된다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729124158722.png" width="300px">

매우 쉬운 네모로직이므로 풀고 'gogo'버튼을 눌러 보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729124254460.png" width="200px">

이름 입력 받는 창이 나오게 되고 입력하고 'write'를 누르면,

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729124346824.png" width="400px">

내 아이피와 아이디, 그리고 answer값이 표시된다.

여기서 answer값은 소스코드를 보면 알 수 있다.

```html
<td onclick="if(this.style.background!='black') { this.style.background='black'; kk._1.value=1; } else { this.style.background='white';kk._1.value=0; }" >&nbsp;</td>
```

소스코드에서 일부 코드만 가져왔다. 코드를 살펴보면 onclick이 일어났을 때, 배경을 검정으로 바꾸고, kk._1.value의 값을 1로 셋팅한다. 이런식으로 각 칸마다 _1, _2, _3 식으로 _25 까지 코딩되어 있다.

```js
function go()
{
    var answer="";
    for(i=1;i<=25;i++) { answer=answer+eval("kk._"+i+".value"); }
    kk._answer.value=answer;
    kk.submit();
}
```

그리고 나서 스크립트 부분에서 kk._[숫자].value를 확인하면서 해당 값들을 answer로 저장한다.

각 칸들은 클릭되면 1, 아니면 0으로 셋팅 되어 있다.

따라서 answer라는 값이 '1010100000011100101011111 ' 이렇게 출력 된것이다.

처음 answer값이 flag인줄 알았으나 아니였다.

그리고 접근한 것이 설마 이 값이 2진수로 생각하고 변환해 보기도 하였지만 딱히 얻어낼 수 있는 결과가 없었다.

삽질이 계속 되다가 사용자로 입력 받는 부분을 공략해보자는 생각으로 네모로직을 풀고 이름을 입력 받는 부분을 SQL Injection해 보기 시작했다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729125433928.png" width="500px">

크롬 개발자 모드에서 살펴보니 사용자 입력받는 페이지에서 넘겨지는 value값이 지정되어 있었다.

그래서 이부분을 수정하여 SQL Injection을 하였다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729125822366.png" width="600px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729125919666.png" width="500px">
'no hack'이라는 문자열이 출력됬다.

필터링이 걸려 있는 것 같다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729130042082.png" width="600px">

'=' 에 필터링이 되어 있나 싶어 'true'로 바꿨더니 똑같이 'no hack' 이라고 떳다.

이번엔 'or'를 바꿔 보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729130255990.png" width="600px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 03/image-20180729130329941.png" width="600px">

'or'를 '||' 으로 바꾸고 보내니 문제가 풀렸다.

**flag : new_sql_injection**