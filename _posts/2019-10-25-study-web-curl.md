---
title: "웹 취약점 점검 도구로써의 cURL 사용법"
tags: [curl, web, HTTP Header, HTTP Method, manual, telnet, 웹취약점분석]
author: eli_ez3r
key: 20191025
modify_date: 2019-10-25
article_header:
  type: cover
  image:
    src:
---

![Image](http://eliez3r.synology.me/assets/img/study/web/curl/1.jpg){:.border.rounded}

## 0x00. 개요

[cURL]( https://curl.haxx.se/ )은 FTP, 고퍼(Gopher), HTTP, HTTPS, SCP, LDAP등 다양한 통신규악을 지원하는 명령줄 방식의 데이터 전송도구이다. "cURL"이라는 이름은 "URL을 보다.(See URL)"라는 의미에서 지었다고 한다.

웹 취약점 분석중에서는 서버응답 헤더 분석, 파일 업로드 자동화 등에 활용할 수 있다.

-----

## 0x01. 설치

리눅스 시스템에서 다음 명령어를 통해 간단하게 설치할 수 있다.

```
apt-get install curl -y
```

-----

## 0x02. 사용법

cURL을 이용한 취약점 점검 대상은 [메모지.com](http://www.memozee.com)으로 하였다.



### (1) 서버의 HTTP 응답헤더 확인 (-I)

![Image](http://eliez3r.synology.me/assets/img/study/web/curl/2.png){:.border.rounded}

`-I` 옵션으로 서버응답 헤더를 점검할 수 있다.



### (2) 서버가 허용하는 HTTP Method 목록 확인 (-v -X OPTIONS)

![Image](http://eliez3r.synology.me/assets/img/study/web/curl/3.png){:.border.rounded}

> Apache 웹서버에서는 /icons, /cgi-bin 등의 디렉토리가 Alias로 지정되기 때문에 기본 웹 경로(DocumentRoot)와는 다른 결과를 보여주기도 한다. 또는 기본 웹 경로에서는 OPTIONS, TRACE 등의 HTTP Method가 허용되지 않으나 이러한 디렉토리에서는 허용되기도 하므로 취약점 점검 시에 고려해야 한다.



### (3) TRACE Method 점검 (-v -X TRACE)

![Image](http://eliez3r.synology.me/assets/img/study/web/curl/4.png){:.border.rounded}

`-v -X TRACE` 옵션으로 HTTP 요청헤더가 응답 내용에 출력되는지 확인한다.

-----



## 0x03. cURL이 없는 시스템의 대안 (telnet, nc)

MS 윈도우 등과 같이 cURL이 설치되어 있지 않은 경우에는 telnet, nc로 대체할 수도 있다. 다만, 이러한 경우에는 HTTP 규약을 이해하고 있어야 한다.

### (1) telnet을 이용한 TRACE 메소드 점검

![Image](http://eliez3r.synology.me/assets/img/study/web/curl/5.png){:.border.rounded}

telnet을 이용한 TRACE 메소드 점검방법이다. 녹색 부분은 사용자가 직접 입력해 주어야 하는 내용이다.

-----

