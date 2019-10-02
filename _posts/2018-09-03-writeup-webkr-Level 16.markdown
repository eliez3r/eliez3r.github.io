---
layout: post
title:  "[Web.kr]Level 16"
subtitle:   "[Web.kr]Level 16"
categories: Webhacking.kr(Old)
tags:
- Wargame
- webhacking.kr(Old)
- Write-up
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 16/4E5A62F4-0F2D-4C82-86D1-14DF4F8AC772.png" width="400px">

키보드 를 누르면 '*' 문자가 생기고 마우스를 갖다 대면 지워진다...



뭥미...? 😕 

```html
<html>
<head>
<title>Challenge 16</title>
<body bgcolor=black onload=kk(1,1) onkeypress=mv(event.keyCode)>
<font color=silver id=c></font>
<font color=yellow size=100 style=position:relative id=star>*</font>
<script> 
document.body.innerHTML+="<font color=yellow id=aa style=position:relative;left:0;top:0>*</font>";

function mv(cd)
{
kk(star.style.posLeft-50,star.style.posTop-50);
if(cd==100) star.style.posLeft=star.style.posLeft+50;
if(cd==97) star.style.posLeft=star.style.posLeft-50;
if(cd==119) star.style.posTop=star.style.posTop-50;
if(cd==115) star.style.posTop=star.style.posTop+50;
if(cd==124) location.href=String.fromCharCode(cd);
}


function kk(x,y)
{
rndc=Math.floor(Math.random()*9000000);
document.body.innerHTML+="<font color=#"+rndc+" id=aa style=position:relative;left:"+x+";top:"+y+" onmouseover=this.innerHTML=''>*</font>";
}

</script>
</body>
</html>
```



mv 함수를 자세히 보니 cd의 값을 비교하며 작업을 수행하는데 마지막 부분에 cd가 124이면 특정 사이트로 이동 되는것 같다.

cd값을 어떻게 변경해야 할까?

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 16/F7E5C168-3DA2-4B5D-BA10-B86CECCA696F.png" width="400px">



python으로 확인해보니 ascii값 124는 '|'(파이프) 였다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 16/09CCD8A1-F443-4CCE-AF33-BA4742144046.png" width="400px">



크롬 개발자모드의 콘솔에서 mv함수를 강제로 불러와 인자값으로 124를 넘겨줘도 풀릴것 같다.



#### flag : webhacking.kr

