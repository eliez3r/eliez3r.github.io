---
title: "웹 취약점 점검 도구로써의 cURL 사용법"
tags: [curl, HTTP header, HTTP method, manual, telnet, 웹취약점분석]
author: eli_ez3r
key: 20190003
category: study
date: 2019-10-25 01:00:00 +0900
modify_date: 2021-02-26
article_header:
  type: cover
  image:
    src: 
---

![Image](http://eliez3r.synology.me/assets/img/study/web/curl/1.jpg)

## 0x00. 개요

[cURL]( https://curl.haxx.se/ )은 FTP, 고퍼(Gopher), HTTP, HTTPS, SCP, LDAP등 다양한 통신규악을 지원하는 명령줄 방식의 데이터 전송도구이다. download/upload 모두 가능하며 HTTP, HTTPS, FTP, LDAP, SCP, TELNET, SMTP, POP3 등 중요 프로토콜을 지원하며 Linux/Unix 계열 및 Windows 등 중요 OS에서 구동된다.

"cURL"이라는 이름은 "URL을 보다.(See URL)"라는 의미에서 지었다고 한다.

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



## 0x04. 주요 옵션

curl [options...] <url> 형식으로 사용하면 된다.

| short | long                 | 설명                                                         | 비고                                                         |
| ----- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -k    | --insecure           | https 사이트를 SSL certificate 검증없이 연결한다.            | wget 의 --no-check-certificate 과 비슷한 역할 수행           |
| -l    | --head               | HTTP header 만 보여주고 content 는 표시하지 않는다           |                                                              |
| -D    | --dump-header <file> | <file> 에 HTTP header 를 기록한다.                           |                                                              |
| -L    | --location           | 서버에서 [HTTP 301 ](http://en.wikipedia.org/wiki/HTTP_301)이나 [HTTP 302](http://en.wikipedia.org/wiki/HTTP_302) 응답이 왔을 경우 redirection URL 로 따라간다.--max-redirs 뒤에 숫자로 redirection 을 몇 번 따라갈지 지정할 수 있다. 기본 값은 50이다 | curl -v daum.net 을 실행하면 결과값으로 다음과 같이 HTTP 302 가 리턴된다.<br />< HTTP/1.1 302 Object Moved<br/>< Location: http://www.daum.net/<br />-L 옵션을 추가하면 [www.daum.net](http://www.daum.net) 으로 재접속하여 결과를 받아오게 된다. |
| -d    | --data               | HTTP Post data                                               | FORM 을 POST 하는 HTTP나 JSON 으로 데이타를 주고받는 REST 기반의 웹서비스 디버깅시 유용한 옵션이다 |
| -v    | --verbose            | 동작하면서 자세한 옵션을 출력한다.                           |                                                              |
| -J    | --remote-header-name | 어떤 웹서비스는 파일 다운로드시 [Content-Disposition Header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html) 를 파싱해야 정확한 파일이름을 알 수 있을 경우가 있다. -J 옵션을 주면 헤더에 있는 파일 이름으로 저장한다. | curl 7.20 이상부터 추가된 옵션                               |
| -o    | --output FILE        | curl 은 remote 에서 받아온 데이타를 기본적으로는 콘솔에 출력한다. -o 옵션 뒤에 FILE 을 적어주면 해당 FILE 로 저장한다. (download 시 유용) |                                                              |
| -O    | --remote-name        | file 저장시 remote 의 file 이름으로 저장한다. -o 옵션보다 편리하다. |                                                              |
| -s    | --silent             | 정숙 모드. 진행 내역이나 메시지등을 출력하지 않는다. -o 옵션으로 remote data 도 /dev/null 로 보내면 결과물도 출력되지 않는다 | HTTP response code 만 가져오거나 할 경우 유리                |
| -X    | --request            | Request 시 사용할 method 종류(GET, POST, PUT, PATCH, DELETE) 를 기술한다. |                                                              |

---



## 0x05. 주요 진단 명령어 예시

### Basic Auth

ID/PW가 필요한 사이트의 경우 -u(--user) 옵션을 이용하여 인증할 수 있다.

```
curl -u admin:password -ks https://[URL] -v
```

-u : `<user:password>` 형태로 입력하며, http 헤더에 Authorization값에 Basic으로 Base64되어 입력됨

```sh
GET /artifactory/api/builds/ HTTP/1.1
> Host: [HOST]
> Authorization: Basic YWNjZXNzLWFkbWluOnBhc3N3b3Jk
> User-Agent: curl/7.55.1
> Accept: */*
```



명령행 json 처리기인 `jq`를 연동해서 서버의 Json 응답을 보기 좋게 파싱

```
curl -u admin:password -ks https://[URL] -v | jq .
```



### Bearer Token Auth

OAuth나 JWT 등에 사용하는 Bearer Token을 사용하려면 -H 옵션뒤에 `Authorization: Bearer [TOKEN]`을 추가하며 `[TOKEN]`은 실제 토큰으로 변경하면 된다.

```
curl -v -L  -X POST -H 'Accept: application/json' -H 'Authorization: Bearer 12345' 'https://www..example.com/api/myresource'
```



### POST 전송

-X POST 옵션을 추가하거나 -d(--data) 옵션을 지정하면 기본값으로 POST로 설정한다.
POST 데이터는 `param1=value1&param2=value2...` 형식으로 전달한다.

```
curl -d "first_name=Bruce&last_name=Wayne&press=%20OK%20" http://posttestserver.com/post.php
```



### URL Encoding

데이터에 공백이나 특수 문자가 있을 경우 URL Encoding을 적용해야 한다.

7.18.0버전 이전의 cURL을 사용할 경우 공백을  `+`로 변환해서 전송해야 하지만, 이후 버전부터는 `--data-urlencode` 옵션을 사용하면 된다.

```shell
curl --data-urlencode "first_name=Bruce" --data-urlencode "last_name=Wayne" --data-urlencode "press= OK " http://posttestserver.com/post.php
```

