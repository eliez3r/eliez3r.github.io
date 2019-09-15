---
layout: post
title:  "[Web.kr]Level 06"
subtitle:   "[Web.kr]Level 06"
categories: Wargame[webkr]
tags:
- Wargame
- webhacking.kr
- Write-up
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 06/B2AE219F-AC72-4011-AA48-75D1CB29645F.png" width="400px">

```php+HTML
<?php 
if(!$_COOKIE[user]) 
{ 
    $val_id="guest"; 
    $val_pw="123qwe"; 

    for($i=0;$i<20;$i++) 
    { 
        $val_id=base64_encode($val_id); 
        $val_pw=base64_encode($val_pw); 
    } 
    $val_id=str_replace("1","!",$val_id); 
    $val_id=str_replace("2","@",$val_id); 
    $val_id=str_replace("3","$",$val_id); 
    $val_id=str_replace("4","^",$val_id); 
    $val_id=str_replace("5","&",$val_id); 
    $val_id=str_replace("6","*",$val_id); 
    $val_id=str_replace("7","(",$val_id); 
    $val_id=str_replace("8",")",$val_id); 

    $val_pw=str_replace("1","!",$val_pw); 
    $val_pw=str_replace("2","@",$val_pw); 
    $val_pw=str_replace("3","$",$val_pw); 
    $val_pw=str_replace("4","^",$val_pw); 
    $val_pw=str_replace("5","&",$val_pw); 
    $val_pw=str_replace("6","*",$val_pw); 
    $val_pw=str_replace("7","(",$val_pw); 
    $val_pw=str_replace("8",")",$val_pw); 

    Setcookie("user",$val_id); 
    Setcookie("password",$val_pw); 

    echo("<meta http-equiv=refresh content=0>"); 
} 
?> 

<html> 
<head> 
<title>Challenge 6</title> 
<style type="text/css"> 
body { background:black; color:white; font-size:10pt; } 
</style> 
</head> 
<body> 

<? 
$decode_id=$_COOKIE[user]; 
$decode_pw=$_COOKIE[password]; 

$decode_id=str_replace("!","1",$decode_id); 
$decode_id=str_replace("@","2",$decode_id); 
$decode_id=str_replace("$","3",$decode_id); 
$decode_id=str_replace("^","4",$decode_id); 
$decode_id=str_replace("&","5",$decode_id); 
$decode_id=str_replace("*","6",$decode_id); 
$decode_id=str_replace("(","7",$decode_id); 
$decode_id=str_replace(")","8",$decode_id); 

$decode_pw=str_replace("!","1",$decode_pw); 
$decode_pw=str_replace("@","2",$decode_pw); 
$decode_pw=str_replace("$","3",$decode_pw); 
$decode_pw=str_replace("^","4",$decode_pw); 
$decode_pw=str_replace("&","5",$decode_pw); 
$decode_pw=str_replace("*","6",$decode_pw); 
$decode_pw=str_replace("(","7",$decode_pw); 
$decode_pw=str_replace(")","8",$decode_pw); 

for($i=0;$i<20;$i++) 
{ 
    $decode_id=base64_decode($decode_id); 
    $decode_pw=base64_decode($decode_pw); 
} 
echo("<font style=background:silver;color:black>&nbsp;&nbsp;HINT : base64&nbsp;&nbsp;</font><hr><a href=index.phps style=color:yellow;>index.phps</a><br><br>"); 
echo("ID : $decode_id<br>PW : $decode_pw<hr>"); 

if($decode_id=="admin" && $decode_pw=="admin") 
{ 
    @solve(6,100); 
} 
?> 
</body> 
</html> 
```

초기 id = 'guest', pw = '123qwe'로 설정되어 있고, base64로 20번 인코딩 된 다음 치환 알고리즘을 통과하고 쿠키에 값을 담고 페이지를 리셋한다.

그리고 쿠키값에서 id와 pw를 가져와 치환 되었던 값들을 다시 복구 하고, base64 20번 디코딩 후, id와 pw가 'admin' 이면 문제가 풀린다.

따라서 쿠키값 안에 id와 pw를 admin을 base64로 인코딩 후, 똑같은 치환 과정을 거친 값을 넣어주면 될 것 같다.

> 소스코드에서 base64로 인코딩하고 숫자들을 일정 특수문자로 바꾸는데, 어쩌피 다시 원상태로 치환 시키게 되고, base64는 본래 특수문자가 없는 암호문이여서 그대로 넣어주면 된다.

![B2CC6AEA-A950-4077-8D1E-E59D9ECDC339](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 06/B2CC6AEA-A950-4077-8D1E-E59D9ECDC339.png)



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 06/FE670E40-13F7-4DFA-BDEB-79AC98531466.png" width="400px">

