---
title: "ARM의 Pipeline"
tags: [arm, pipeline, assembly]
author: eli_ez3r
key: 20191114
modify_date: 2019-11-14
article_header:
  type: cover
  image:
    src: /assets/img/arm_logo.png
---

-----

ARM Core는 메모리에 있는 코드를 읽어오고, 해석하고, 실행한다. 이를 `Fetch` - `Decode` - `Execute`라고 한다.

여기서 각 단계 별로 CPU clock을 1cycle 소모한다고 가정해보자.

<img src="http://eliez3r.synology.me/assets/img/study/arm/pipeline/1.png" width="600px">

2개의 어셈블리 명령어가 수행된다고 할 때, 총 6cycle이 필요하게 된다. (사진 상단 파란색). 이때 fetch의 입장으로 볼 때, 처음 fetch가 된 후로 나머지 2cycle동안 쉬게 된다.

ARM은 효율적으로 명령을 실행하기 위해 Pipeline을 도입하였다.(사진 하단 연두색). Fetch가 일어나고 다음 명령어를 다음 cycle에서 미리 Fetch한다. 이로인해 4cycle만으로 동일한 결과를 얻어 성능이 향상하게 된다.

Pipeline은 기본적으로 3단계로 이루어져 있으며 세부적으로 5단계, 8단계, 13단계로 나눌 수 있다.

- ARM7 : 3단계
- ARM9 : 5단계
- ARM11 : 8단계
- CortexA8 : 13단계

-----







