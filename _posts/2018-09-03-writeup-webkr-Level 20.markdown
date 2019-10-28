---
title: "[Web.kr]Level 20"
tags: [wargame, webhacking.kr(old), writeup]
author: eli_ez3r
key: 20180903
modify_date: 2018-09-03
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 20/image-20180726143450823.png" width="500px">

```php+HTML
<input type=button value="Submit" onclick=ck()>
<script>
function ck()
{
if(lv5frm.id.value=="") { lv5frm.id.focus(); return; }
if(lv5frm.cmt.value=="") { lv5frm.cmt.focus(); return; }
if(lv5frm.hack.value=="") { lv5frm.hack.focus(); return; }
if(lv5frm.hack.value!=lv5frm.attackme.value) { lv5frm.hack.focus(); return; }
lv5frm.submit();
}
</script>
```

해당 페이지의 주요 코드를 보면 위와 같다. id, cmt, hack값을 입력받고 submit을 한다.



<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 20/image-20180726143708550.png" width="500px">

 입력하고 "Submit"하니 "Wrong"이라고 뜬다. time limit:2 인것 보니 2초 안에 해야 되는 것같다.

그래서 크롬 console을 이용하여 다음 코드를 한번에 입력하였다.

```js
javascript:(lv5frm.id.value='1'); 
javascript:(lv5frm.cmt.value='1');
javascript:(lv5frm.hack.value=lv5frm.attackme.value);
javasecript:ck();
```

![image-20180726144251890](http://eliez3r.synology.me/assets/img/writeup/webkr/Level 20/image-20180726144251890.png)

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 20/image-20180726144230888.png" width="500px">

