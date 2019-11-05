---
layout: post
comments: true
title: 서버리스 환경에서의 docker 선능 극복 사례
feature-img: "assets/img/jeju.jpg"
categories : [IdeaLog]
tags : [Docker, Serverless, MSA]
---

## Serverless

최종 서버 자원 까지도 공유하겠다.

### Function As a Service

함수만 만들어서 넣으면 클라우드 서버가 구동되고 해당 서버에서 함수가 돌고 끝나면 없어지는 방식인듯.

- like WebFiddle IDE?

### apache openwhisk

- 서버리스 오픈소스 플랫폼
- 사용자 함수를 Action이라는 개념으로 활용
