---
title: "[Web.kr]Level 15"
tags: [Wargame, webhacking.kr(Old), Write-up]
article_header:
  type: cover
  image:
    src: 
---

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 15/DC7DAE26-507F-442B-88B9-060A5EB66AE6.png" width="500px">

level 15 버튼을 누르면 "Access_Denied"라는 알림창이 뜨고 들어가 지지 않는다.



fiddler를 통해 캡처를 해보았다.

<img src="http://eliez3r.synology.me/assets/img/writeup/webkr/Level 15/B85FA2F7-55F2-40EF-8903-6863DE5ACAD8.png" width="400px">

패킷에 브레이크 포인트를 걸고 하나씩 보내보니



"password is off_script"라고 뜨고 전 페이지로 리다이렉트 되었다.

```html
<html>
<head>
<title>Challenge 15</title>
</head>
<body>
<script>
alert("Access_Denied");
history.go(-1);
document.write("password is off_script");
</script>
</body>
```



#### flag : off_script