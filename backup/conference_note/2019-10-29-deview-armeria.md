---
comments: true
title: 희승이형과 아메리아 (아메리아 아님 아르메리아임)
feature-img: "assets/img/jeju.jpg"
categories : [IdeaLog]
tags : [Ameria, Java, gRPC]
---

## 아메리아 [아르메리아]

마이크로 서비스 프레임워크.

### api 지원

- haProxy 지원.
- 웹 지원
- gRPC 지원.
- Thrift 지원.

- 각 api서비스 통합 가능..??

### 비동기, reactive

데이터 사이즈가 무지막지 하게 클 때 reactive로 해결 가능함.

### HTTP이외의 네트워크 커스텀 콜 툴

documentation service

### 아메리아, 프레임워크 설계

- 스프링 부트
- 쥬스
- 대거
- gRPC, REST, Thrift 선택가능.

스프링과다름!

### 아메리아 모니터링 시스템

- 프로메테우스
- 헬스체크
- 허니컴, brave 서비스  (집킨 수집 가능)
- doc 서비스

### fail over

- 클라이언트 사이드 로드 벨런싱.


### 헬스체크

- http2를 이용한 커넥션 헬스체크 기능
- 반대로 상태 변경이 일어 날 때에만 응답을 주게 함.
