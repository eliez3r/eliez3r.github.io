---
layout: post
title:  "[Web.kr]Level 04"
subtitle:   "[Web.kr]Level 04"
categories: Webhacking.kr(Old)
tags:
- Wargame
- webhacking.kr(Old)
- Write-up
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 04/637DD772-03E3-4FB0-BB79-4D2A60193796.png" width="500px">

대소문자알파벳 + 숫자 + '='가 있는 것으로 보아 base64인것 같다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 04/4DC341D9-E7EB-4F0A-8D81-3D89E425E7DD.png" width="400px">



c4033bff94b567a190e33faa551f411caef444f2

flag 인줄 알았더니 .. 아니였다.

이렇게 쉬울리가 없지.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 04/E4AE081B-53C0-4BDD-99C4-3DD282568E7B.png" width="400px">

사이즈를 보니 40바이트이다.



구글에 "40바이트 암호화"라고 치고 확인해보니



SHA1의 암호화 출력값 길이는 160비트(40바이트)라고 한다. ✌️

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 04/5FA5C789-0C84-4772-B611-D688BEC0DB0B.png" width="500px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 04/8F7767DE-1DA8-4C03-83C4-49F53D930880.png" width="500px">

SHA1 decrypt 사이트에서 값을 넣어 복호화 하니 또 다시 40바이트 짜리 암호문이 나왔다. 그래서 한번더 복호화 하니 flag값을 얻을 수 있었다.



#### flag : test