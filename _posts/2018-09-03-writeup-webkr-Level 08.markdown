---
title: "[Web.kr]Level 08"
tags: [Wargame, webhacking.kr(Old), Write-up]
article_header:
  type: cover
  image:
    src: 
---

```php
<?
$agent=getenv("HTTP_USER_AGENT");
$ip=$_SERVER[REMOTE_ADDR];

$agent=trim($agent);

$agent=str_replace(".","_",$agent);
$agent=str_replace("/","_",$agent);

$pat="/\/|\*|union|char|ascii|select|out|infor|schema|columns|sub|-|\+|\||!|update|del|drop|from|where|order|by|asc|desc|lv|board|\([0-9]|sys|pass|\.|like|and|\'\'|sub/";

$agent=strtolower($agent);

if(preg_match($pat,$agent)) exit("Access Denied!");

$_SERVER[HTTP_USER_AGENT]=str_replace("'","",$_SERVER[HTTP_USER_AGENT]);
$_SERVER[HTTP_USER_AGENT]=str_replace("\"","",$_SERVER[HTTP_USER_AGENT]);

$count_ck=@mysql_fetch_array(mysql_query("select count(id) from lv0"));
if($count_ck[0]>=70) { @mysql_query("delete from lv0"); }


$q=@mysql_query("select id from lv0 where agent='$_SERVER[HTTP_USER_AGENT]'");

$ck=@mysql_fetch_array($q);

if($ck)
{ 
    echo("hi <b>$ck[0]</b><p>");
    if($ck[0]=="admin")
        {
            @solve();
            @mysql_query("delete from lv0");
        }
}
if(!$ck)
{
    $q=@mysql_query("insert into lv0(agent,ip,id) values('$agent','$ip','guest')") or die("query error");
    echo("<br><br>done!  ($count_ck[0]/70)");
}
?>
```

소스 코드가 길므로 문제를 푸는데 중요한 부부만 살펴보도록 하겠다.



```php
$agent=getenv("HTTP_USER_AGENT");
$ip=$_SERVER[REMOTE_ADDR];

$agent=trim($agent);

$agent=str_replace(".","_",$agent);
$agent=str_replace("/","_",$agent);

$pat="/\/|\*|union|char|ascii|select|out|infor|schema|columns|sub|-|\+|\||!|update|del|drop|from|where|order|by|asc|desc|lv|board|\([0-9]|sys|pass|\.|like|and|\'\'|sub/";

$agent=strtolower($agent);

if(preg_match($pat,$agent)) exit("Access Denied!");

```

먼저 agent에는 http_user-agent정보가 담기게 된다.

그리고 trim을 통해 스트링 앞뒤의 공백을 제거하고,

str_replace로 문자를 치환한다. 그리고 strtolower로 문자열들을 소문자로 변경한다.

preg_match를 통해 agent에 해당 패턴들이 있는지 필터링한다.



```php
$q=@mysql_query("select id from lv0 where agent='$_SERVER[HTTP_USER_AGENT]'");

$ck=@mysql_fetch_array($q);

if($ck)
{ 
    echo("hi <b>$ck[0]</b><p>");
    if($ck[0]=="admin")
        {
            @solve();
            @mysql_query("delete from lv0");
        }
}
if(!$ck)
{
    $q=@mysql_query("insert into lv0(agent,ip,id) values('$agent','$ip','guest')") or die("query error");
    echo("<br><br>done!  ($count_ck[0]/70)");
}
```

lv0 테이이블에서 id 레코드를 조회하는데 agent로 조건으로 조회한다.

이때 존재하면 ck에 값이 들어가고, 없으면 if문을 통해 insert 쿼리문이 생성된다.



우리의 공격 포인트는 이부분이다. insert를 통해 레코드를 생성하는데 agent값을 넣을 수 있다.

`insert into lv0(agent,ip,id) values('$agent','$ip','guest')`



먼저 알아야 할 개념은, values를 이용해 값을 넣을때 ','를 이용하여 여러 데이터를 넣을 수 있다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 08/image-20180730133907271.png" width="600px">



이방법을 이용해여 쿼리를 넣게 되면, `eunice','ip','admin'),('agent`  값을 넣게되면

`insert into lv0(agent,ip,id) values('eunice','ip','admin'),('agent','$ip','guest')`

이렇게 쿼리가 완성되게 된다. 그러면 'eunice' 계정은 'admin'이 되게 된다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 08/image-20180730134322534.png" width="700px">

fiddler로 User-Agent 부분을 수정하여 패킷을 보낸다.

그러면 처음에는 해당 User-Agent가 없으므로 Insert 쿼리를 통해 레코드가 생성이 되게 된다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 08/image-20180730134400578.png" width="700px">

그리고 또다시 접속을 하고, 이번에는 User-Agent를 'eunice'로 수정하여 보내게 되면 문제가 풀리게 된다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 08/image-20180730132805720.png" width="400px">











