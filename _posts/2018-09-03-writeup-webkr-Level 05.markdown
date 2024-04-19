---
title: "[Web.kr]Level 05"
tags: [wargame, webhacking.kr(old), writeup]
author: eli_ez3r
key: 20180903
modify_date: 2018-09-03
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729140333626.png" width="400px">

Login 버튼과 Join 버튼이 존재한다. 소스코드를 보자.

```php
<input type=button value='Login' style=border:0;width:100;background=black;color=green onmouseover=this.focus(); onclick=move('login');>
<input type=button value='Join' style=border:0;width:100;background=black;color=blue onmouseover=this.focus(); onclick=no();>

<script>
    function no()
    {
    	alert('Access_Denied');
    }

    function move(page)
    {
    	if(page=='login') { location.href='mem/login.php'; }
    }
</script>
```

'Login' 버튼을 누르면 mem/login.php로 이동이 되고, 'Join'을 클릭하면 "Access_Denied" 경고창을 띄운다.

'Login' 버튼을 눌러보자.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729140610737.png" width="300px">

로그인 창이 뜨길래 'test' 값을 둘다 넣어 login해보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729140712120.png" width="200px">

admin으로 로그인해야 되는 것 같다. 그래서 admin으로 로그인 해봤다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729140804866.png" width="200px">



그래서 admin으로 로그인 해야 되나 싶어 SQL Injection을 했다. ~~(이는 삽질이였다. 😢)~~

그래서 이것 저것 코드살펴보고 하다가 처음 화면에서 'Login' 버튼을 누르면 mem/login.php 로 이동되는 것을 보고 혹시 디렉토리 노출 취약점이 있나 싶어 접속해보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729141259268.png" width="400px">

'혹시나' 가 '역시나'로 바뀌는 순간이였다. 😏

login.php는 우리가 접속했던 페이지이고, join.php 파일을 열어보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729141531176.png" width="500px">

"access_denied"라는 경고창이 뜬다. join.php 파일을 다운받아서 소스코드를 살펴보았다.

```php
<html>
<title>Challenge 5</title></head><body bgcolor=black><center>
<script>
l='a';ll='b';lll='c';llll='d';lllll='e';llllll='f';lllllll='g';llllllll='h';lllllllll='i';llllllllll='j';lllllllllll='k';llllllllllll='l';lllllllllllll='m';llllllllllllll='n';lllllllllllllll='o';llllllllllllllll='p';lllllllllllllllll='q';llllllllllllllllll='r';lllllllllllllllllll='s';llllllllllllllllllll='t';lllllllllllllllllllll='u';llllllllllllllllllllll='v';lllllllllllllllllllllll='w';llllllllllllllllllllllll='x';lllllllllllllllllllllllll='y';llllllllllllllllllllllllll='z';I='1';II='2';III='3';IIII='4';IIIII='5';IIIIII='6';IIIIIII='7';IIIIIIII='8';IIIIIIIII='9';IIIIIIIIII='0';li='.';ii='<';iii='>';lIllIllIllIllIllIllIllIllIllIl=lllllllllllllll+llllllllllll+llll+llllllllllllllllllllllllll+lllllllllllllll+lllllllllllll+ll+lllllllll+lllll;
lIIIIIIIIIIIIIIIIIIl=llll+lllllllllllllll+lll+lllllllllllllllllllll+lllllllllllll+lllll+llllllllllllll+llllllllllllllllllll+li+lll+lllllllllllllll+lllllllllllllll+lllllllllll+lllllllll+lllll;if(eval(lIIIIIIIIIIIIIIIIIIl).indexOf(lIllIllIllIllIllIllIllIllIllIl)==-1) { bye; }if(eval(llll+lllllllllllllll+lll+lllllllllllllllllllll+lllllllllllll+lllll+llllllllllllll+llllllllllllllllllll+li+'U'+'R'+'L').indexOf(lllllllllllll+lllllllllllllll+llll+lllll+'='+I)==-1){alert('access_denied');history.go(-1);}else{document.write('<font size=2 color=white>Join</font><p>');document.write('.<p>.<p>.<p>.<p>.<p>');document.write('<form method=post action='+llllllllll+lllllllllllllll+lllllllll+llllllllllllll+li+llllllllllllllll+llllllll+llllllllllllllll
+'>');document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name='+lllllllll+llll+' maxlength=5></td></tr>');document.write('<tr><td><font color=gray>pass</font></td><td><input type=text name='+llllllllllllllll+lllllllllllllllllllllll+' maxlength=10></td></tr>');document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');}
</script>
</body>
</html>
```

난독화가 되어있다. http://jsbeautifier.org 에 접속하여 소스코드를 이쁘게 보이도록 먼저 수정하였다.

```php
<html>
<title>Challenge 5</title>
</head>

<body bgcolor=black>
    <center>
        <script>
            l = 'a';
            ll = 'b';
            lll = 'c';
            llll = 'd';
            lllll = 'e';
            llllll = 'f';
            lllllll = 'g';
            llllllll = 'h';
            lllllllll = 'i';
            llllllllll = 'j';
            lllllllllll = 'k';
            llllllllllll = 'l';
            lllllllllllll = 'm';
            llllllllllllll = 'n';
            lllllllllllllll = 'o';
            llllllllllllllll = 'p';
            lllllllllllllllll = 'q';
            llllllllllllllllll = 'r';
            lllllllllllllllllll = 's';
            llllllllllllllllllll = 't';
            lllllllllllllllllllll = 'u';
            llllllllllllllllllllll = 'v';
            lllllllllllllllllllllll = 'w';
            llllllllllllllllllllllll = 'x';
            lllllllllllllllllllllllll = 'y';
            llllllllllllllllllllllllll = 'z';
            I = '1';
            II = '2';
            III = '3';
            IIII = '4';
            IIIII = '5';
            IIIIII = '6';
            IIIIIII = '7';
            IIIIIIII = '8';
            IIIIIIIII = '9';
            IIIIIIIIII = '0';
            li = '.';
            ii = '<';
            iii = '>';
            lIllIllIllIllIllIllIllIllIllIl = lllllllllllllll + llllllllllll + llll + llllllllllllllllllllllllll + lllllllllllllll + lllllllllllll + ll + lllllllll + lllll;
            lIIIIIIIIIIIIIIIIIIl = llll + lllllllllllllll + lll + lllllllllllllllllllll + lllllllllllll + lllll + llllllllllllll + llllllllllllllllllll + li + lll + lllllllllllllll + lllllllllllllll + lllllllllll + lllllllll + lllll;
            if (eval(lIIIIIIIIIIIIIIIIIIl).indexOf(lIllIllIllIllIllIllIllIllIllIl) == -1) {
                bye;
            }
            if (eval(llll + lllllllllllllll + lll + lllllllllllllllllllll + lllllllllllll + lllll + llllllllllllll + llllllllllllllllllll + li + 'U' + 'R' + 'L').indexOf(lllllllllllll + lllllllllllllll + llll + lllll + '=' + I) == -1) {
                alert('access_denied');
                history.go(-1);
            } else {
                document.write('<font size=2 color=white>Join</font><p>');
                document.write('.<p>.<p>.<p>.<p>.<p>');
                document.write('<form method=post action=' + llllllllll + lllllllllllllll + lllllllll + llllllllllllll + li + llllllllllllllll + llllllll + llllllllllllllll +
                    '>');
                document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name=' + lllllllll + llll + ' maxlength=5></td></tr>');
                document.write('<tr><td><font color=gray>pass</font></td><td><input type=text name=' + llllllllllllllll + lllllllllllllllllllllll + ' maxlength=10></td></tr>');
                document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');
            }
        </script>
</body>
</html>
```

그나마 알아보기 쉽게 수정되었다. 이제 편집기에 대치 기능을 통해서 소스코드를 수정하였다.

```js
a1b1b1b1b1b1b1b1b1b1a = oldzombie;
a99a = document.cookie;
if (eval(a99a).indexOf(a1b1b1b1b1b1b1b1b1b1a) == -1) {
    bye;
}
if (eval(document.'U'+'R'+'L').indexOf(mode+'='+1) == -1) {
    alert('access_denied');
    history.go(-1);
} else {
    document.write('<font size=2 color=white>Join</font><p>');
    document.write('.<p>.<p>.<p>.<p>.<p>');
    document.write('<form method=post action='+join.php+'>');
    document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name=' + id + ' maxlength=5></td></tr>');
    document.write('<tr><td><font coaor=gray>pass</font></td><td><input type=text name=' + pw + ' maxlength=10></td></tr>');
    document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');
}
```

이렇게 수정되었고, 조금 더 수정을 하면

```javascript
if (eval(document.cookie).indexOf(oldzombie) == -1) {
    bye;
}
if (eval(document.'U'+'R'+'L').indexOf(mode+'='+1) == -1) {
    alert('access_denied');
    history.go(-1);
} else {
    document.write('<font size=2 color=white>Join</font><p>');
    document.write('.<p>.<p>.<p>.<p>.<p>');
    document.write('<form method=post action='+join.php+'>');
    document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name=' + id + ' maxlength=5></td></tr>');
    document.write('<tr><td><font coaor=gray>pass</font></td><td><input type=text name=' + pw + ' maxlength=10></td></tr>');
    document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');
}
```

이렇게 정리가 되었다.

처음 if문 부터 살펴보면 document.cookie에 `oldzombie`라는 쿠키가 있어야 한다.

다음 if문을 보면 url에 `mode=1` 가 있어야 한다.

위 두가지를 설정하고 다시 접속해 보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729142057566.png" width="400px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729142130490.png" width="200px">

회원가입할 수 있는 폼 페이지가 불러졌다. 이제 	admin으로 계정을 만들려고 했다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729142228262.png" width="200px">

admin 계정이 이미 존재한다고 떴다.

여기서 별에별 계정 다 만들고 삽질을 했다. 🤔

결론은 id를 입력받는 길이가 '5'로 정해져 있는데 개발자모드로 이를 수정하고 id를 'admin(공백)'으로 생성한다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729142415762.png" width="100px">

가입에 성공하였다.

그리고 다시 login.php로 접속하여 admin으로 비밀번호는 자신이 만든 값으로 로그인 한다.

이는 로그인 시 id값을 5자리만 확인하는 취약점 때문인것 같다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 05/image-20180729142557323.png" width="400px">

