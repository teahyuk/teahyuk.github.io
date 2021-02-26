---
comments: true
title: 쿠팡 백엔드 상품 추천 시스템 사례
feature-img: "assets/img/jeju.jpg"
categories : [IdeaLog]
tags : [Hadoop,Hbase,Logstore,Datastore]
---

## 쿠팡 추천 시스템 구축 사례

모델 과 서비스 분리.

### 추천 모델 에 의존된 플랫폼, 서비스

모델이 곧 서비스가 됨.

- 모델 변경에 따라 서비스 재시작, 버전 업 등등 느려짐.

### 모델과 서비스 분리

검색엔진 사용.
검색에 관련한 서비스 모델로 활용함. (질의 응답 방식)

#### 서비스 정의 파트

쿼리 튜닝 및 쿼리 적용 방식으로 추천 서비스 개발함.

### Learning to Rank

ML을 활용한 쿼리 튜닝 방식.???

- 추천 쿼리를 갖고 올린 추천 시스템 내부에서의 CTR(클릭 수)나, 유저 반응을 보고 점차 학습하여 쿼리를 변형하는 방식
