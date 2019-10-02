---
layout: post
title:  "[Web.kr]Level 10"
subtitle:   "[Web.kr]Level 10"
categories: Webhacking.kr(Old)
tags:
- Wargame
- webhacking.kr(Old)
- Write-up
---

![image-20180728232337075](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 10/image-20180728232337075.png)

```html
<a id=hackme style="position:relative;left:0;top:0" onclick="this.style.posLeft+=1;if(this.style.posLeft==800)this.href='?go='+this.style.posLeft" onmouseover=this.innerHTML='yOu' onmouseout=this.innerHTML='O'>O</a><br>
```

소스코드에서 중요한 부분만 가져왔다.

해석하자면 id가 hackme이고 'O' 라는 문자의 위치는 (0,0)이다.

그리고 클릭할 때 마다 left좌표가 1씩 증가한다. 즉, 오른쪽으로 1씩 움직이게 된다.

if문을 통해 left좌표가 800이 되면 ?go=800이 url에 추가되면서 이동한다.

그리고 'O'문자 위에 마우스를 올리면 'yOu'라고 표시된다.



처음 이문제를 풀때 크롬에서 풀었다. 클릭해도 아무런 반응이 없고, url에 ?go=800을 넣으면 'no hack'이라는 문자열이 출력되어서 쿼리문을 우회해서 푸는 문제인줄 알고 30분간 엄청난 삽질...💢♨️



혹시나 하고 Internet Explorer로 실행하니 클릭할 때마다 정상적으로 움직인다. (하아...💢)

~~이번일을 꼭 기억해두자. 다음부터 삽질 안하길....😭~~



Internet Explorer로 한번 클릭할 때 1이 아니라 800씩 움직이도록 수정하고 클릭 한방으로 문제를 풀었다.

(이렇게 쉬운걸....😫)

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 10/image-20180728232302009.png" width="200px">