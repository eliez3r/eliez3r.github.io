---
layout: post
title:  "[Webkr]Level 01"
subtitle:   "[Webkr]Level 01"
categories: writeup
tags: webkr level1

---

# Level 1

## keyword : cookie 변조

```php+HTML
<?
if(!$_COOKIE[user_lv])
{
SetCookie("user_lv","1");
echo("<meta http-equiv=refresh content=0>");
}
?>
<html>
<head>
<title>Challenge 1</title>
</head>
<body bgcolor=black>
<center>
<br><br><br><br><br>
<font color=white>
---------------------<br>
<?

$password="????";

if(eregi("[^0-9,.]",$_COOKIE[user_lv])) $_COOKIE[user_lv]=1;

if($_COOKIE[user_lv]>=6) $_COOKIE[user_lv]=1;

if($_COOKIE[user_lv]>5) @solve();

echo("<br>level : $_COOKIE[user_lv]");

?>
<br>
<pre>
<a onclick=location.href='index.phps'>----- index.phps -----</a>
</body>
</html>
```

user_lv이없으면 SetCookie로 인해 user_lv를 1로 설정한다.

eregi에 의해 user_lv가 0~9가 아니여도 user_lv가 1로 설정된다.

또, user_lv가 6이상이면 user_lv가 1로 설정한다.

user_lv가 5보다 크면 문제가 풀리게 된다.



내용을 종합하면 user_lv가 5보다 크고, 6보다 작은 값이면 된다.

<img src="/assets/img/writeup/webkr/Level 01/image-20180728234027339.png" width="400px">

크롬 확장플러그인을 이용하여 쿠키 값을 5.5로 바꾸면 된다.