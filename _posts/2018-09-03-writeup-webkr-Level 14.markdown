---
layout: post
title:  "[Web.kr]Level 14"
subtitle:   "[Web.kr]Level 14"
categories: Webhacking.kr(Old)
tags:
- Wargame
- webhacking.kr(Old)
- Write-up
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 14/5F9BF4BF-6EB1-4E74-B763-6DF6BCF4F9F9.png" width="400px">

```php+HTML
<html>
<head>
<title>Challenge 14</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
</style>
</head>
<body>
<br><br>
<form name=pw><input type=text name=input_pwd><input type=button value="check" onclick=ck()></form>
<script>
function ck()
{
    var ul=document.URL;
    ul=ul.indexOf(".kr");
    ul=ul*30;
    if(ul==pw.input_pwd.value) 
    { 
        alert("Password is "+ul*pw.input_pwd.value); 
    }
    else { alert("Wrong"); }
}
</script>
</body>
</html>

```

![F1C654AA-1C82-4944-BFB3-C7288B41545E](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 14/F1C654AA-1C82-4944-BFB3-C7288B41545E.png)

check버튼을 누르면 ".kr"까지 인덱스 크기를 구하고, 그 크기에 30을 곱한 값이 ul에 담기게 된다.

텍스트 박스에 값과 해당 값이 같으면 패스워드가 출력된다.

즉, 510을 넣고 버튼 누르면 끝.

![72584E58-4BA5-473C-887D-DE4EA5521185](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 14/72584E58-4BA5-473C-887D-DE4EA5521185.png)



#### flag : 260100

