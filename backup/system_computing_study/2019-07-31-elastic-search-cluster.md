---
comments: true
title: 클러스터
feature-img: "assets/img/jeju.jpg"
categories : [Services]
tags : [Elastic Stack, Elastic Search]
---

## 기본개념 클러스터 요약

클러스터 기본 개념

> 컴퓨터 클러스터(영어: computer cluster, 문화어: 콤퓨터 클라스터)는 여러 대의 컴퓨터들이 연결되어 하나의 시스템처럼 동작하는 컴퓨터들의 집합을 말한다. 클러스터의 구성 요소들은 일반적으로 고속의 근거리 통신망으로 연결된다. 서버로 사용되는 노드에는 각각의 운영 체제가 실행된다. 컴퓨터 클러스터는 저렴한 마이크로프로세서와 고속의 네트워크, 그리고 고성능 분산 컴퓨팅용 소프트웨어들의 조합 결과로 태어났다. 

엘라스틱서치에서 클러스터는 노드의 모음으로 커다란 하나의 엘라스틱 서치 시스템의 가장 큰 단위로 볼 수 있다.

### 노드

- 각 컴퓨터, 또는 프로그램 단위의 각 프로세스인스턴스(단일 jvm실행 단위)로써 물리적으로 하나의 pc에 있을 수도 있고, 다른 pc에 있을 수도 있다.

#### 종류

1. master node : 인덱스 생성 관리등을 위한 노드 CRUD I/O처리, 최 앞단 노드임.
2. data node : 데이터 관리를 위한 노드, 검색 집계등의 처리. (I/O 보통 Public API 막음.)
3. ingest node : 전처리 파이프라인을 위한 노두
4. coordinating only node : 집계등을 실행할때의 데이터 계산 리소스를 사용할만한 노드

### 샤드

- 실제로 엘라스틱서치에서 사용중인 루신엔진 1개의 단위로 노드당 여러개의 샤드를 들고있고, 하나의 인덱스가 여러개의 샤드에 분산되서 저장 된다.

- 루씬 자체에서 내부적으로 세그먼트라는 단위의 데이터로 쪼개서 작업을 한다.

### 데이터 저장 방법

- 루신이 NIO를 사용하고 java NIO는 시스템 버퍼를 사용하기 떄문에 jvm 힙 메모리 외에 메모리를 더 사용 할 수 있다.
  - 보통 전체 메모리의 반 정도만 jvm 힙 메모리로 설정하라고 하는 이유

1. Refresh : 검색 가능해진다, 물리저장 x, log는 자체적으로 하는중
1. Flush : 5초에한번인가 한다. 물리저장하고 log지움.
1. Optimaize API : 파편화된 세그먼트를 merge한다, 검색속도 향상됨.

|엘라스틱 서치|루씬|NIO|
|-|-|-|
|Refresh|Flush|Memory Buffer|
|Flush|Commit|Disk IO|
|Optimize API|Segment Merge|Disk IO|


## 클러스터링 방법

기본적으로 각 노드 프로세스 실행 시 설정을 통해 cluster.name을 가진 노드가 있으면 자동으로 클러스터링하여 합친다. 이를 Discovery라고 한다.

### [Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-discovery-settings.html)

하나의 클러스터에서 새로운 노드가 추가되거나 마스터노드를 잃어버려서 새로운 마스터노드 선출 과정이 진행 되는 과정.

> multicast방식은 네트워크 위험성으로 지워졌고 discovery.seed_hosts를 통해 추가된 호스트 내에서만 추가된 노드가 있는지 주기적으로 확인한다.

### 클러스터링 bootstrap

클러스터 최초 시작시 선출 할 마스터 노드 후보를 정하라. (`cluster.initial_master_nodes`)

- `{node.name}`, `{IP:HOST}`, `{IP주소}`를 사용 할 수 있다.

마스터노드 후보군이 결정되어있으면 다른 노드들이 추가 실행되고 정지되는 과정들에 알아서 클러스터 유지관리를 수행하므로 안전해진다.

자동 부트스트랩 모드는 실제 운영시에는 사용하지말자!

자동 부트스트랩 모드를 해제하는 설정 3가지

- `discovery.seed_providers`
- `discovery.seed_hosts`
- `cluster.initial_master_nodes`

> **!주의 : 마스터노드는 데이터 노드와 같이 `data_path`에 접근 가능하게 하는것이 좋다. </br>이는 `data_path`가 각 노드들이 재시작되는 사이에 클러스터 상태가 저장되는 위치라서 그렇다.**

### [동적 노드 생성 및 제거](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-discovery-adding-removing-nodes.html)

#### 노드 추가

기본적으로 discovery세팅값에 따른 정의된 호스트 내부에서 같은 클러스터명을 가지고 노드를 추가실행하면 설정에 따라 자동적으로 discovery기능을 이용해 노드가 추가된다.

마스터노드 추가 시에도 discovery 설정에 따라 추가된다.

#### 노드 제거

##### 마스터노드 제거

마스터 노드후보군 제거는, 마스터노드 선출 로직에 영향을 미치므로 한꺼번에 제거하면 동작하지 않는다.

기본적을 마스터노드 후보군은 홀수개 이며, 후보선출을 위해 최소 선출 경계값은 늘 (n/2+1/2) 개 이므로 한꺼번에 (n/2-1/2) 개를 지우면 클러스터가 멈추게 된다.

충분히 최소 선출 경계값을 마스터노드 후보군의 갯수에 맞게 줄일 수 있도록 충분한 시간간격을 두고 1~2개씩 줄이도록 한다.

마스터노드 개수는 홀수여야하지만 짝수개일 시에는 특정 후보 노드의 투표권을 제외시켜 홀수 투표 로직이 잘 동작 할 수 있게끔 할 수 있다.

``` sh
# Add node to voting configuration exclusions list and wait for the system
# to auto-reconfigure the node out of the voting configuration up to the
# default timeout of 30 seconds
POST /_cluster/voting_config_exclusions/node_name

# Add node to voting configuration exclusions list and wait for
# auto-reconfiguration up to one minute
POST /_cluster/voting_config_exclusions/node_name?timeout=1m
```

또는 해당 명령을 이용하면 안전하게 여러개의 마스터노드 후보군 노드를 중지 시킬 수도 있다.

최대 투표권 제외 노드 개수는 설정값 `cluster.max_voting_config_exclusions`로 제한 할 수 있고 기본값은 10개다.

> **!주의 : 투표권이 제외된 노드의 투표권을 다시 올리려면 해당 노드를 끄고 설정을 변경하는것이 안전하다. 그냥 설정을 지우면 정상동작하지 않는다.</br>
즉 투표권 제외 api는 마스터 노드를 지울때 말고는 사용하지말라!**

### 클러스터 상태

클러스터 상태 관리는 마스터노드가 한다.

각 노드들이 마스터노드로 상태 갱신 및 조회를 요청하고 마스터노드는 클러스터상태를 관리하고 정리하는 역활이다.

상태 주기는 설정 가능하며 기본적으로 30초다.

마스터노드는 각 노드들로 heartbeat를 보내 노드가 잘 살아있는지 주기적으로 관리하고 상태를 변경한다.

마스터노드가 죽은게 확인되면 클러스터는 종료된다.

### [클러스터 레벨에서의 샤드 위치 조정](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cluster.html)

`cluster.routing.*` 설정으로 설정 가능

#### [샤드레벨 설정](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shards-allocation.html)

- 클러스터 상에서 샤드를 어느노드에 어떻게 분배 할 지 설정 가능하다.


#### [디스크 레벨 설정](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/disk-allocator.html)

- 해당 노드의 디스크 상태에 따른 샤드 분배 설정을 할 수 있다.


#### [샤드위치 인식 설정](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html)

- 샤드 위치가 어디있는지 확인하는 작업은 꽤나 무거운 작업이다. 이를 위한 설정을 할 수 있다.


#### [샤드 할당 필터링](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-filtering.html)

- 샤드 할당시 제외할 노드들 설정 가능하다.


#### [기타 설정](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/misc-cluster.html)

클러스터를 위한 나머지 기타 설정들이다.

- `cluster.blocks.read_only` : 클러스터 api 를 다이나믹하게 세팅 할지 못할지 설정한다.
  - 주의 : 이것만으로 클러스터 세팅을 제한하려하지 마라. read-write로 재설정 가능하므로 애초에 cluster-update-sttings api에 접근 못하도록 따로 추가 제한을 둬라.

- `cluster.max_shards_per_node` : 한 노드당 최대 샤드 개수이다. 복제던 아니던 상관없다
- `cluster.metadata.*` : 클러스터의 메타데이터를 임의의 key-value 형식으로 설정 가능, key는 `cluster.metadata.*` 로 설정
- `Persistent Tasks Allocationsedit` : 클러스터가 재시작되더라도 실행되야할 긴 작업에 대해서 `영속성 작업`이라는 이름으로 따로 설정 가능함.

## 다중 클러스터

### [원격 클러스터](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-remote-clusters.html)

클러스터 복제(DR을 위한.?, x-pack 기능이다) 또는 다중 클러스터 검색을 위해 단방향으로 원격 클러스터를 세울 수 있다.

통신방향은 단방향이다. 오직 다른클러스터들 조회만 하기위한 클러스터.</br>
gateway 노드라는 제한된 노드들만 사용가능함.



### [다중 클러스터 검색](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html)

상위의 원격 클러스터에서만 사용 가능하다.

#### 문법

- `/{cluster_name}:{index_name}/_search`

## [클러스터 백업](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html)

- 클러스터 전체를 snapshot 뜬 후 restore하는 방식으로 백업이 가능하다.
- 단순히 data파일들을 복제하는 방식으로는 백업이 절대 안됨.
- snapshot기능으로만 백업이 가능하며 restore기능으로만 복구가 가능하다.
- 이방식을 통해서 업그레이드(마이그레이션) 도 가능.
  - 물론 해당방식이 아닌 [무중단 업그레이드](https://www.elastic.co/guide/en/elastic-stack/7.2/upgrading-elastic-stack.html) 방법이 따로 있다.

### 버전 호환성

전버전으로 snapshot을 뜬 백업본을 다음 버전으로 restore 가능하다.(7.x는 안되는듯?)

- A snapshot of an index created in 5.x can be restored to 6.x.
- A snapshot of an index created in 2.x can be restored to 5.x.
- A snapshot of an index created in 1.x can be restored to 2.x.