---
comments: true
title: Elastic search 데이터 구조
feature-img: "assets/img/jeju.jpg"
categories : [Services]
tags : [Elastic Stack, Elastic Search]
---

## 데이터 구조

1. 인덱스 : 예전엔 RDBMS의 Database와 같게보았으나 요즘은 테이블과 매치됨
2. 타입 : Deprecated됨, index한개당 type 하나
3. 도큐먼트 : Row
4. 필드 : 각 도큐먼트의 키? 라고 볼 수 있음.

### 역색인 방식

검색을 위해 indexing이 완료 되면 데이터는 먼저 각 매핑 된 방식에 따라 각 키워드 별로 해당되는 document의 id가 저장되는 방식으로 저장됨.

서로 겹치지 않는 형태소, 키워드 등으로 저장되면 2배 이상 씩 용량을 잡아먹지만 겹치는 키워드들이 많을 수 있으므로 모든 데이터에 대해서 2배 이상씩 잡아먹지 않음.

또 매핑 방식, 또는 저장 방식에 따라서 rawData 저장 안 할 수도 있음.

### plugin들

- 2.x에서 지원하던 plugin들이 보안상의 이유로 없어진 경우가 있다.
- 이제는 유료로 x-pack에서 지원하는게 많으므로 그곳을 찾아봐도 된다.

- head는 쓰지말고, bigdesk를 standalone버전으로 사용 해보면 좋을 듯.
> [bigdesk(ElasticStack용 APM이라보심 될듯)](http://asuraiv.blogspot.com/2017/05/elasticsearch-5x-head-bigdesk.html)

- x-pack에 각 cluster의 샤드 상태 확인 가능
> [x-pack(cluster-status)](https://www.elastic.co/guide/en/x-pack/6.2/watch-cluster-status.html)

## 데이터 수집, indexing

### 공통
- `_`가 붙으면 메타데이터(내장필드) 또는 elasticSearch쪽 명령을 사용하게 된다.
- url param에 `pretty`를 붙이면 json이 이쁘게 정렬되서 나옴
- `@`를 이용하면 파일을 입력값으로 사용 가능하다.

### 데이터 입력

- RestAPI를 사용한다
- Post로 create, PUT으로 업데이트를 한다,,, 고 나와있지만 둘 다 된다 구분 없다.
- 둘다 없으면 입력 있으면 업데이트로 대체한다.
- 업데이트에는 remove에걸리는 시간이 상당하며,메타데이터(_version)이 증가한다.
    - 이전데이터를 확인할 순 없지만 용량은 잠시 늘어날 수 있다.
- document id를 입력안하면 임의의 id가 생성된다.
    - 단 PUT으로는 생성이 안된다. POST로 해야한다.
- 업데이트는 사실상 삭제 후 재 입력이라서 값이 높다.

### 데이터 삭제

- 삭제는 즉시 삭제가 아니다.
- 마킹 후 검색 안되게 막는다.
- 메타 데이터는 남아있어서 같은 id로 다시 입력하게되면 _version이 올라간다.

### 데이터 업데이트

- '_update'명령을 사용한다.
- 내부 동작 방식은 'GET'으로 가져와서 확인 후 script결과에 맞게 다시 'POST'하는 방식으로 동작한다.
- script로 MVEL 문법을 사용해서 변경 가능하다.
    - elastic 6.x 이상부터는 painless라는 문법 사용

### bulk

- 대용량 데이터를 한꺼번에 insert할 때 필요하다.
- 메타데이터한줄 다음 실제 데이터 한줄 이런 식으로 동작한다.
- doc개수는 1000~5000개가 좋고 10,000 개 넘어가면 오류가능성이 있다.
- udp를 통해 bulk로 받을 수도 있다.

---

# 매핑

indexing을 위한 매핑 방식을 정의 하는 방법론.

- index 생성 과 함께 매핑 설정 가능
- 매핑 설정 안해도 자동 설정 됨.
- 매핑에 필드 추가는 가능하지만 변경 삭제는 불가능함.
- mapping 삭제를 시도하면 데이터가 다 날라간다고 책에서 설명한다
    - 하지만 DELETE 메소드 자체가 응답 안되도록 에러를 날리게됨.

## 내장필드 매핑

- `_`로 시작하는 엘라스틱서치에서 각 document의 메타데이터와 같은 내부 기능들을 설명한 필드들이다.

### `_id`

- document id를 나타냄

#### 옵션

1. index : 색인 여부 (default false)
2. store : 저장 여부 (default false)
3. path : 내부 필드 중 한가지로 자동 id 매핑 (deprecated)
    - 원인은 같은 index내부에서 다른 type의 같은 내부 필드 path를 id로 지정 했을 때에 aggregation에서 에러가 날 수 있다는 것이다.
    - 6.x이후로 같은 index의 여러 type은 제거되었지만, _id속성이 document내용에 의존성을 갖는다는 것 자체가 좋지 않다고 여겨져 그대로 폐기된 상태이다.

### `_source`

- 원본데이터 저장되는 내장필드

#### 옵션

1. enabled : 전체 raw data 저장여부
2. includes : 저장 포함할 필드
3. excludes : 저장 제외할 필드

### `_all`

- 검색할 대상 필드 지정용 내장필드

#### 옵션

1. enabled : `_all` 사용여부
2. 각 properties 의 필드 중 inculde_in_all : 색인 사용 여부

### `_analyzer`

- 분석기사용 내장필드

### `_timestamp`

- document저장 된 시간의 타임스탬프

#### 옵션

1. enabled : 사용여부
2. store : 저장 여부

### `_ttl`

- 데이터 유지 시간 설정
- 시간이 지나면 자동으로 사라짐

#### 옵션

1. enabled : 사용여부 (default false)
2. default : 유지할 시간 (d,m,h,s,ms,w 를 이용한 string)

## 데이터 타입

|type|내용|
|---|---|
|text|analyze 가 더 필요한 분석되지 않은 </br>날것 그대로의 string필드|
|keyword|논리적이든 어떤 모양으로든 분석이 완료된 </br>더이상의 분석이 필요없는 string필드|
|long|숫자|
|float|부동소수점 숫자|
|double|double|
|date|날짜|
|boolean|bool값|
|binary|binary타입|
|range|각 primitive타입 간 범위,</br> 검색시 유용하다.|
|object|json자체가 스키마가없는 여러단계의 데이터로 활용 가능하다.</br> 매핑도 그냥 doc뎁스와 같이 설정 하면 된다.|
|geo-point|지도 좌표|
|geo-shape|지도에서의 모양까지 포함된 geo-point를 포함하고있는 특별한 타입|
|ip|ip주소|
|token|주로 멀티필드로 사용되며 토큰화된 수 저장함|

### 다중필드

하나의 필드에서 여러 타입을 지정 할 수 있다. </br>
이러면 여러 필드의 색인에 저장되여 여러 방식으로 검색이 가능하다

### 필드 복사

필드 복사도 가능하다. 방식은 필요할 때 봐서 확인하여 사용





