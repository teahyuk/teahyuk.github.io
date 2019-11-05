---
layout: post
comments: true
title: 우형 스프링 배치 세미나
feature-img: "assets/img/jeju.jpg"
categories : [IdeaLog]
tags : [Spring, Batch]
---

## spring batch 기본

### 실행

job, step 등으로 cli App으로도, java 함수 호출로도 사용 가능.

### JobScope & StepScope

jobscope stepscope 는 lazy binding이라 동적 생성가능

#### jobParameter

cli Option으로 실행시 설정 가능함.

jobParameter 는 int,long,String,Date는 가능하지만 LocalDateTime은 안됨.

jobParameter 지원의 미약함을 value어노테이션의 스코프(파라미터 아닌 메소드에서도 붙일 수 있음)을 통해 컨버터 래퍼클래스등을 만들어놓고 jobscope를 해당 클래스에 붙여서 type safe하게 그리고 컨버팅하는 과정을 모아놓고 처리함

## 활용

### 쿼츠와 스프링 배치의 차이점

#### 쿼츠

스케줄 프레임 워크로, 어떤 프로그램이 됐든 리눅스 상에서 스케줄링해서 프로그램을 실행 할 수 있는 기능을 제공함.

#### 스프링 배치

단순 배치 프로그램 생성 기능,

배치프로그램(cli) 실행 프로그램에 최적화.

### 스케줄링 방식(스프링 배치 실행 스케줄링) 들.

spring mvc 이용 rest로 실행 (원격지에서 스케줄해서 실행).

spring batch admin deprecated  => 이제 사용 안함.

quartz admin => 많이 사용하는 방식.

쿼츠대신
cI tool??
jenkins???

### jenkins

원래는 java기반의 opensource CI툴.

#### 갖고 있는 기능

- 스케줄링
- 파이프라인
- 로깅
- 단독실행

##### jenkins Pipeline

스프링 배치에서의 순서가 있는 다중 작업을 하나의 job 여러 step으로 개발하지 않음.

1job에 1step으로 사용하게끔 하고, jenkins의 pipeline을 이용해서 순사작업이 진행되도록 수정.

이렇게 할 시에 테스트용도로 어느 한 작업만 실행 가능.

#### 멱등성

- 멱등성(함수의 항상성)

일정한 파라미터로 실행하면 일정한 결과값이 나올 수 있도록 사용자가 제어하지 못하는 상황이나 상태를 참조하여 실행 결과가 달라지는 작업(또는 함수)를 만들면 안됨.

ex)날짜조차도 파라미터로 받아라 (날짜에 따른 다른 결과른 내는 작업은 멱등성을 해침)

- 기대 효과
  - 테스트 할때 신뢰성 있고 용이한 테스트 결과를 기록 가능함.

젠킨스에 java LocalDateTime 조차도 파라미터로 지정해서 실행 할 수 있게 하는 함수가 있음.

### TEST 에 대하여

배치작업은 interact한 프로그램이 아닌 한번 실행하면 내부적으로 알아서 동작해야 하는 프로그램 으로써 QA가 일일히 확인하기 어렵기 때문에, TC가 필수적임.

배치 작업은 각 작업당 java instance 가 구동되므로, 보통 ConditianalOnProperty 등등을 조건에 추가하여 Spring 구동 시 파라미터를 받아 구동에 필요한 bean만 업로드하도록 설정 할 수 있음.

문제는 여기서 발생하는데, ConditianalOnProperty로 인해 Junit 테스트시 매번 바뀌는 Context로 인해 Spring Context자체가 제구동되면서 테스트 속도가 현저히 느려짐.

결국 배치작업에서 필요 없는 bean생성으로 인한 메모리 사용량의 증가 정도로는 성능에 크게 영향을 미치지 않으며, 실시간 작업도 아니라서 괜찮을 것으로 판단, ContitianlOnProperty를 전부 제거함.

Junit 테스트 시에 한번에 모든 bean 업로드된 뒤로 한번 올라간 Context로 전체 테스트 수행하므로 테스트시간 단축함.

Junit 테스트 중 MockBean등 필요 할 때에는 어쩔 수 없이 Spring Context가 재 로드되지만 해당 MockBean이 필요한 테스트는 많지 않아서 무시함.

