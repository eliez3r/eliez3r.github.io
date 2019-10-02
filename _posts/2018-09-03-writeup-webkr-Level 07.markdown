---
layout: post
title:  "[Web.kr]Level 07"
subtitle:   "[Web.kr]Level 07"
categories: Wargame[webkr]
tags:
- Wargame
- webhacking.kr
- Write-up
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180729233649699.png" width="150px">

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180729233740588.png" width="400px">



'auth' 버튼을 누르니 "Access_Denied!" 경고창이 뜬다.

소스코드를 살펴보자.

```php+HTML
<!--
db에는 val=2가 존재하지 않습니다.

union을 이용하세요
-->
<?
$answer = "????";

$go=$_GET[val];

if(!$go) { echo("<meta http-equiv=refresh content=0;url=index.php?val=1>"); }

$ck=$go;

$ck=str_replace("*","",$ck);
$ck=str_replace("/","",$ck);

echo("<html><head><title>admin page</title></head><body bgcolor='black'><font size=2 color=gray><b><h3>Admin page</h3></b><p>");

if(eregi("--|2|50|\+|substring|from|infor|mation|lv|%20|=|!|<>|sysM|and|or|table|column",$ck)) exit("Access Denied!");

if(eregi(' ',$ck)) { echo('cannot use space'); exit(); }

$rand=rand(1,5);

if($rand==1)
{
    $result=@mysql_query("select lv from lv1 where lv=($go)") or die("nice try!");
}

if($rand==2)
{
    $result=@mysql_query("select lv from lv1 where lv=(($go))") or die("nice try!");
}

if($rand==3)
{
    $result=@mysql_query("select lv from lv1 where lv=((($go)))") or die("nice try!");
}

if($rand==4)
{
    $result=@mysql_query("select lv from lv1 where lv=(((($go))))") or die("nice try!");
}

if($rand==5)
{
    $result=@mysql_query("select lv from lv1 where lv=((((($go)))))") or die("nice try!");
}

$data=mysql_fetch_array($result);
if(!$data[0]) { echo("query error"); exit(); }
if($data[0]!=1 && $data[0]!=2) { exit(); }

if($data[0]==1)
{
    echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=
        alert('Access_Denied!')><p>");
    echo("<!-- admin mode : val=2 -->");
}

if($data[0]==2)
{
    echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=
        alert('Congratulation')><p>");
    @solve();
} 
?>
```



코드가 너무 길어서 차근차근 살펴보도록 하자.

```html
<!--
db에는 val=2가 존재하지 않습니다.

union을 이용하세요
-->
```

힌트 부분이다. val=2가 존재하지 않는다고 한다. (없다고 하니 있는지 확인해야지 ㅋㅋ😏)



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180729234506533.png" width="500px">

없는거 맞나 싶다. ㅋㅋㅋㅋ 다음 코드를 살펴보자.



```php
$answer = "????";
$go=$_GET[val];

if(!$go) { echo("<meta http-equiv=refresh content=0;url=index.php?val=1>"); }

$ck=$go;

$ck=str_replace("*","",$ck);
$ck=str_replace("/","",$ck);
```

val의 값은 go에 저장되고 go의 값이 없으면 index.php?val=1 페이지를 불러온다.

go의 값이 있다면, ck에 go의 값을 저장한다.

그리고 나서 ck에서 '*', '/' 를 제거한다.



```php
echo("<html><head><title>admin page</title></head><body bgcolor='black'><font size=2 color=gray><b><h3>Admin page</h3></b><p>");

if(eregi("--|2|50|\+|substring|from|infor|mation|lv|%20|=|!|<>|sysM|and|or|table|column",$ck)) exit("Access Denied!");

if(eregi(' ',$ck)) { echo('cannot use space'); exit(); }
```

그리고나서 페이지를 출력해주고, ck의 값에서 위 문자들을 필터링 한다.

필터링 되는 값중에서 '2', 'substring', '%20', 'or' 도 있다.

그리고 공백이 있으면 'cannot use space' 라고 출력되고 필터링된다.

val=2의 값을 만들어야 될거 같은데, 숫자 2도 필터링 되어있다.



```php
$rand=rand(1,5);

if($rand==1)
{
    $result=@mysql_query("select lv from lv1 where lv=($go)") or die("nice try!");
}

if($rand==2)
{
    $result=@mysql_query("select lv from lv1 where lv=(($go))") or die("nice try!");
}

if($rand==3)
{
    $result=@mysql_query("select lv from lv1 where lv=((($go)))") or die("nice try!");
}

if($rand==4)
{
    $result=@mysql_query("select lv from lv1 where lv=(((($go))))") or die("nice try!");
}

if($rand==5)
{
    $result=@mysql_query("select lv from lv1 where lv=((((($go)))))") or die("nice try!");
}
```

1~5의 램덤 수를 rand에 저장시킨다.

그리고 rand에 값에 따라 go를 감싸는 괄호의 갯수가 정해진다.



```php
$data=mysql_fetch_array($result);
if(!$data[0]) { echo("query error"); exit(); }
if($data[0]!=1 && $data[0]!=2) { exit(); }

if($data[0]==1)
{
    echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=
        alert('Access_Denied!')><p>");
    echo("<!-- admin mode : val=2 -->");
}

if($data[0]==2)
{
    echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=
        alert('Congratulation')><p>");
    @solve();
} 
```

쿼리문을 통해 출력된 값(lv값)들이 data에 저장되고 그 값이 2이면 문제가 풀린다.



이제 차근차근 풀어 나가보자.

먼저 UNION에 대해 알 필요가 있다.

> MySQL 등의 RDBMS에서 사용하는 Union연산자는 여러 테이블에 존재하는 같은 성격의 값을 한번의 쿼리로 추출할 수 있도록 돕는다.



예를들면 다음과 같다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180729235548415.png" width="300px">



서로 다른 레코드가 담긴 테이블이 2개가 있다. 이때 Union을 이용하면, 한번의 쿼리로 다른 테이블의 값을 추출할 수 있다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180729235818938.png" width="300px">

위 사진과 같이 한번의 쿼리로 test2와 test3에서 id의 값을 추출하였다.



다음 예시도 살펴보자.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180730000349195.png" width="400px">

test3의 테이블에서 where절을 이용하여 'guest'를 추출하고 있다.

역시 Union과 select를 이용하면 어떻게 되는지 살펴보자.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180730000514308.png" width="500px">

select 'admin' 을 붙였더니 'admin'도 출력되었다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180730000934101.png" width="400px">

실제 select 명령어는 입력한 값을 그대로 출력해 준다.



‼️ 여기서 중요한 점은 우리는 data[0]값을 '2'로 만들어야 된다는 점. 

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180730001138181.png" width="500px">

따라서 위 사진 예제와 같이 where절 앞부분에 거짓정보를 넣고 select로 입력한 값이 나오게 하면 될 것같다.



해당 문제의 쿼리문은 다음과 같다. `select lv from lv1 where lv=($go)`

$go 부분에 `-1) union select (2` 라고 넣게 되면 lv에 -1이란 값은 없으므로 거짓이 되고 '2'라는 값이 추출된다.



이제 넣어야 할 쿼리는 알았으니 필터링만 우회하면 된다.

일단 공백은 %0a를 넣으면 된다.

> %0a는 ascii코드로 라인피드(line feed), 즉 개행문자



그리고 숫자 2는 간단하다. '3-1'을 넣으면 된다.



![image-20180730002537484](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 07/image-20180730002537484.png)

읭? 뭥미....



구글링 주위 지인에게 물어본 결과... 문제 에러라고 한다... (허무하네😫)

