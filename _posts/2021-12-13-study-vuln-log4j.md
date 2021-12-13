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





-----

출처: [ESTsecurity](https://blog.alyac.co.kr/4341)

-----

[^1]:취약점이 알려지지 않거나, 관련 패치가 배포되지 않은 시기
[^2]: 취약점 대응 패치가 배포된 시기
