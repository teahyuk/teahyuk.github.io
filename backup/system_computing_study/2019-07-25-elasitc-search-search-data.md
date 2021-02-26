---
comments: true
title: Elastic search 데이터 검색
feature-img: "assets/img/jeju.jpg"
categories : [Services]
tags : [Elastic Stack, Elastic Search]
---

## 방식

1. URI질의
2. request body 형식
3. 멀티테넌시 (여러인덱스 동시검색)

- `_search` 사용

## URI 검색

- uri param 사용
- 멀티테넌시 지원 
    - delimeter = `,`
    - ex) `server:9200/index1,index2/_search?q={keyword}&pretty`

### `q` 

- `q={keyword}`
- `{field명}:{질의어}`

### `df`

- `q={keyword}&df={field명}`

### `default_operator`

- 띄어쓰기로 구분된 여러 검색 명령어를 `or` 또는 `and` 로 선택
  - default : `or`
- `q={keyword1}%20{keyword2}&default_operater=AND`

### `explain`

- db explain과 같은듯

### `_source`

- raw data 컨텍스트임
- 검색할 때 쓰면 raw data를 보여줄지 여부 결정함
  - default : true
- `_source=false`

### `fileds`

- projection 같음
- 표시할 필드들만 추가
- `fields={field1},{field2}`

### `sort`

- 정렬
- default : `asc`
- text나 다른 tokenize된 데이터들의 경우 정렬이 먼저 진행 된다
  - 맨 앞의 토큰을 가지고 하는것이아닌 각 토큰들 중 가장 앞에 sort된 부분에 대해서 정렬한다
  - 역색인이라 당연한 일인 듯 하다.
- `sort={field}:desc`

### `timeout`

- 제한시간
- 제한시간에 걸리면 그때까지 검색한 애들만 결과로 보여준다.

### `from`

- mariadb의 offset과 같음

### `size`

- mariadb limit와 같음
- default : 10개

### `search_type`

- 검색 방법 지정

1. query_then_fetch - 전체 샤드 검색 후 결과 출력
2. query_and_fetch - 샤드별로 검색되는대로 출력
3. dfs_query_then_fetch - 검색어 빈도수 먼저 계산 (전체샤드)
4. dfs_query_and_fetch - 검색어 빈도 수 먼저 계산 (각 샤드별)
5. count - 갯수만 출력
6. scan - scroll 과 같이 사용되며 scroll_id로 나중에 결과 출력

- `search_type=query_and_fetch`
- `search_type=scan&scroll=10m` : scan방식으로 10분간 스크롤 살아있게 함.


## RquestBody 검색

- body에 QueryDSL 넣어서 사용한다.
- QueryDSL
  - 엘라스틱 서치에서만 쓰는 Custom 질의어임
  - json형식
  - jpa와 다름

### QueryDSL 구조

#### 위와 같은 구조

- `from` : 위와같음
- `size` : 위와같음
- `fields` : 위와같음
- `sort` : 위와같고 array에 대해 추가지원함
  - array필드의 경우 min,max,avg,sum지원.
  - `ignore_unmapped` sort하려는 필드가 없을때 무시 여부
- `track_scores` : 점수 표시 설정
- `_source` : 위와같음
  - 내부 필드별 설정 가능
  - 필드명에 대해 와일드카드 사용 가능
  - projection 기능을 하게 된다.
- `fields` : `_source`와 같은 projection
  - `_source`와 다르게 결과값이 `fields :{` 로 한번 더 가공되서 나온다.
  - full field명만 가능
  - `partial_fields`
    - `_source`처럼 include, exclude사용
    - 와일드카드 및 부분적 field명 사용 가능
    - 여러 partial선택 가능
  - `fielddata_fields` : 전체 데이터 출력
- `highlight` : 강조표시
  - `<em></em>`태그로 강조됨
  - 태그변경 가능

## QueryDSL

- 쿼리는 일반적으로 전문검색에 사용되고 필터는 yes/no조건의 바이너리 구분에 사용된다.
- 쿼리는 점수가 있다 filter는 없다.
- 쿼리결과는 캐싱 되지 않는다, 필터는 된다
- 쿼리는 응답이 느리고 필터는 빠르다.

### 텀, 텀즈 쿼리

- `term`, `terms`
- 텀 이란 분석 이후 토큰화 된 각 조각들.
- 분석 이후 토큰화된 각 텀 에 대한 쿼리.
- 텀즈로 여러 텀을 and 해서 쿼리함
  - text타입이나 기타 타입들의 경우 각 필드가 토큰화되서 나누어진 이후에 각각의 쿼리들을 and연산해서 검색 할 수 있다.
- `minimum_shuold_match`
  - 최소 포함되어야 하는 텀의 개수

### 매치, 다중 매치 쿼리

- `match`
- 검색 시에 분석을 거친다.
- 띄어쓰기하면 or로 합쳐서 분석 (default)
- `operator`
  - `and` or `or`
- `analyzer`
  - 분석기 사용 종류
  - ex) `whitespace` : 띄어쓰기로 토크나이징 하는 분석기
- 이후는 안맞아서 여기 참조
  - https://www.elastic.co/guide/en/elasticsearch/reference/7.3/query-dsl-match-query.html

### 문자열 쿼리

- `query-string`
- url에서 `q`와 같음
  - `{field}:{data}`
- `default-field`
  - 필드 선택
- `default-operator`
  - 검색어 띄어쓰기 합침조건

## Term Suggestion API

- 단어 제안 api

#### 용도

- 오타 수정이나, 완성문장을 위해 사용됨.

### 종류

- Term Suggest API: 추천 단어 제안
- Completion Suggest API: 자동완성 제안
- Phrase Suggest API: 추천 문장 제안
- Context Suggest API: 추천 문맥 제안

### 한글

- 한글은 결합문자라서 다른 알고리즘과 방식이 필요.
  - ex) ㄱ(U+3131) + ㅏ(U+314F) = 가(U+AC00)

  ## [search_as_you_type](https://www.elastic.co/blog/elasticsearch-7-2-0-released)

### [search_as_you_type](https://www.elastic.co/blog/elasticsearch-7-2-0-released)

- 7.2 버전에서 새로 추가됨
- Shingle Token Filter 를 이용하여 기존 필드 이외에 3개의 필드를 더 추가함
    - my_field._2gram
    - my_field._3gram
    - my_field._index_prefix
    #### [Shingle Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-shingle-tokenfilter.html)
        the sentence "please divide this sentence into shingles" might be tokenized into shingles 
        "please divide", "divide this", "this sentence", "sentence into", and "into shingles".  
- 추가된 필드를 이용하여 prefix로 검색 할 수 있고, 중간에서 있는 값도 검색이 가능함
- 공식 문서의 예제에서는 'multi_match' 와 'match_bool_prefix'를 조합하여 사용
    ### [match_bool_prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-bool-prefix-query.html)
    - if message is "**quick brown f**", then query is        
        ```json
        {
            "query": {
                "bool" : {
                    "should": [
                        { "term": { "message": "quick" }},
                        { "term": { "message": "brown" }},
                        { "prefix": { "message": "f"}}
                    ]
                }
            }
        }
        ```