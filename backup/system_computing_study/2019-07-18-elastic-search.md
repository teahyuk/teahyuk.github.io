---
comments: true
title: Elastic Search 용어
feature-img: "assets/img/jeju.jpg"
categories : [Services]
tags : [Elastic Stack, Elastic Search]
---

- 엘라스틱 서치에 대한 용어정리.

## 용어 정리

### Elastic stack

- Elastic Search를 활용하기위한 전체 시스템 서비스들 (Elastic Search, Logstash, Beats, Kibana가 있다)

### 루신

- 아파치 루신 이라는 검색 엔진, 엘라스틱의 엔진 이다.

### 클러스터

- 엘라스틱 서치 시스템의 가장 큰 단위이다. 여러개의 노드로 구성 될 수 있다.

### 노드

- 엘라스틱 서치가 구동되는 프로세스 1개가 1개의 노드이다. 설정 값에 따라 여러 노드로 사용 가능하다
    1. master node : 인덱스 생성 관리등을 위한 노드 CRUD I/O처리, 최 앞단 노드임.
    2. data node : 데이터 관리를 위한 노드, 검색 집계등의 처리. (I/O 보통 Public API 막음.)
    3. ingest node : 전처리 파이프라인을 위한 노두
    4. cross-cluster search node : 여러 클러스터에 참여하는 특별한노드
    5. coordinating only node : 집계등을 실행할때의 데이터 계산 리소스를 사용할만한 노드

### 샤드

- 하나의 인덱스에 따른 데이터 조각, 여러개의 노드에 데이터조각을 뗄 수도 합칠 수도 있다.
- 하나의 인덱스이지만 여러개의 노드에 분산시킬 때 이 샤드를 사용한다고 보면 된다.
- 샤드 = 단일 LUCENE 색인
- LUCENE당 문서 최대 한도
  - 2,147,483,519개`(= INTEGER.MAX_VALUE- 128)`
  - 용량 아님
- 샤드 크기 모니터링 api
  - `{REF}/CAT-SHARDS.HTML[_CAT/SHARDS]`

#### 레플리카

- 샤드 중에 복사본
- failOver를 위한 기능

### 게이트웨이

- 게이트 웨이가 저장소. 문이라 착각하면 안됨
- 엘라스틱 서치의 클러스터의 상태를 저장함

#### 리커버리

- 게이트웨이에 저장된 이전 클러스터의 상태를 재설정하는 과정
- 클러스터가 종료된 후 재실행 될때 작동

### 디스커버리

- 서로 다른 엔드포인트간의 같은 클러스터를 사용하는 노드간 연동을 위해 찾는 과정
  - 멀티캐스트
  - 유니캐스트

### 슬로우 로그

- 설정에 따라 오래걸리는 활동에 로깅하는 기능.

### 도큐먼트

- db로봤을 때 1 row라고보면 됨

### index

- db로 봤을 때 table이나 마찬가지이다.
- 개념은 좀 다르다 index에 명시한 메타데이터를 가지고 내부의 document들을 저장할 때 indexing한다.

### CRUD비교

|RDBMS|CRUD|ElasticSearch|
|-|-|-|
|Insert|Create|POST|
|Select|Read|GET|
|Update|Update|PUT|
|Delete|Delete|DELETE|

### DB구조 비교

|RDBMS|ElasticSearch|
|-|-|
|DataBase|Index|
|Table|Type(Deprecated)|
|Row|Document|
|Column|Field|
|Schema|Mapping|
