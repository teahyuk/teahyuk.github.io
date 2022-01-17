---
comments : true
title: mariadb recursive에 대해
feature-img: "assets/img/sample_feature_img.png"
categories : [DB]
tags : [DB, Recursive, MariaDB]
---

> RDBMS로 무한한 파일경로를 표현 할 방법이 없을 까?  
> 에서 출발한 삽질기.

우선 현재 리눅스는 기본적으로 파일명은 255, 전체 경로 길이는 4K 가 최대길이의 default다.
> https://serverfault.com/questions/9546/filename-length-limits-on-linux

단. 제한 해제가 가능한지는 모르겠다.

윈도우즈는 2016년 win10에서 그동안 제한 걸려있던 260 길이의 경로 제한을 풀어버렸다.
> https://mspoweruser.com/ntfs-260-character-windows-10/

<br/>
<br/>

## 1. mariadb Recursive

- 내가 쓰는 mariadb에는 recursive라는 함수 가 있다(?)

```sql
with recursive file_full_paths (id, path) as (
    ....
)
```

- 친절하게 그래프나 트리구조에서 쓰라고 설명 되어있다.

![heap]({{site.url}}/assets/img/trees_and_graphs.png)

- 단점은 이녀석은 temporal하게만 쓸 수 있다는 것이다.  

> https://mariadb.com/kb/en/library/recursive-common-table-expressions-overview/

## 2. 테이블 생성

### 1. 우선 파일 경로를 나타내는 self referenc Table을 만들어보자.

#### create table

```sql
CREATE TABLE `file_system` (
	`id` INT(11) UNSIGNED NOT NULL,
	`path` VARCHAR(255) NULL DEFAULT NULL,
	`parent` INT(11) UNSIGNED NULL DEFAULT NULL,
	PRIMARY KEY (`id`),
	INDEX `parent_fk` (`parent`),
	FULLTEXT INDEX `path_index` (`path`),
	CONSTRAINT `parent_fk` FOREIGN KEY (`parent`) REFERENCES `file_system` (`id`)
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB;
```

- 우선 기본적으로 리눅스가 한 폴더, 파일 명이 255가 최대이기때문에 path는 255 varchar로.
- 트리구조라서 id과 parent만 있다.  
- 기본적으로 parent가 null 이면 root다.
- path로 검색 할 이슈가 많아보여서 index를 달아줬다. 단 각각의 폴더명은 중복 허용이라 unique는 아니다.

#### dummy data import

```sql
delimiter //

CREATE PROCEDURE dorepeat(pstart INT, pend Int)
  BEGIN
    SET @x = pstart;
    REPEAT 

      SET @x = @x + 1; 

		insert into file_system (id,parent,path) 
		values (@x,
		 		  #floor(RAND()*(@x-1)),
		 		  @x-1,
				  concat( 
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97),
				    char(round(rand()*25)+97)));
		
    UNTIL @x > pend

    END REPEAT;
  END
//

insert into file_system (id,parent,path) values (0,null,'C:/');
insert into file_system (id,parent,path) values (1,null,'D:/');
insert into file_system (id,parent,path) values (2,null,'F:/');
insert into file_system (id,parent,path) values(3,null,'');

CALL dorepeat1(4,10000);
```
- 프로시저는 시작점부터 끝점까지 
랜덤한 이름의 path를 와 직전까지의 id중에 랜덤한 parent를 담은 data를 루프로 돌리면서insert한다.
- 기본적으로 4개의 dummy 루프를 넣는다.
- 빈 스트링은 unix시스템의 root다.
- 컴퓨터가 구려서인지, 루프가 안좋은 것인지  10000개만 넣는데 5분씩걸린다....

### 2. with recursive 사용

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null
)
select * from file_full_paths;
```

- pathSeperator는 '/'로 잡았다.
- recursive안에 union은 루트 로 parent가 null인 row들과 recursive하게 만든 테이블의 결합이다.
- recursive내부에는 parent가 null일 때를 말단으로 잡았다.
- 만약 그래프를 그린다면 중간에 루핑되는 경우를 제거하는 다른 쿼리적 추가기능이 필요할 것이다.

![recursive-result]({{site.url}}/assets/img/full_path_recursive.png)

- 잘된다.

### 3. 성능?

- 나는 heidisql을 쓰는데 하단에 보면 실제 동작쿼리와 속도가 찍힌다.

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null
)
select * from file_full_paths;
/* Affected rows: 0  찾은 행: 29,958  경고: 0  지속 시간 1 쿼리: 0.453 sec. (+ 0.031 sec. network) */
```

- 실제 동작 쿼리야 ui사용안하고 써서 실행시켰으니 같고, 기본적으로 full search가 3만개 row를 기준으로 0.45초가 걸린다...

#### explain

- 실행계획을 한번 봤는데 실행 타입부터가 알수 없는 타입이다.

![recursive-explain]({{site.url}}/assets/img/full_path_recursive_explain.png)

- 결국 실 테스트로 성능 계속 확인하기로 결정.

#### where 사용

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null
)
select * from file_full_paths where path = 'C:';
/* Affected rows: 0  찾은 행: 1  경고: 0  지속 시간 1 쿼리: 0.437 sec. */
```
- where절로 가져오는 row수가 줄어서 그런지 오히려 시간이 0.015초 정도 줄었다.

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null
)
select * from file_full_paths where path like '%f%';
/* Affected rows: 0  찾은 행: 28,004  경고: 0  지속 시간 1 쿼리: 0.438 sec. (+ 0.031 sec. network) */
```
- like로 fullSearch를 돌려도 시간은 비슷하다..

#### order by 

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null
)
select * from file_full_paths order by path;
/* Affected rows: 0  찾은 행: 29,958  경고: 0  지속 시간 1 쿼리: 0.484 sec. (+ 0.031 sec. network) */
```

- order by 는 0.03 초가 더 걸린다.

> 우선 자체적인 테이블 조회 결과로는 0.4초가 기본적으로 깔려 있는 것을 알 수 있다.  
> 이거만으로 성능이 안좋네.. 라며 끝낼 순 있지만 내가원하는건 이게 아니므로 좀 더 해본다.


## 3. 테이블 실제 활용.

- 우선 파일 경로에 관한 컬럼을 추가하고 싶은 건 특정 로그 컬럼이다.

### 1. 데이터 세팅

#### 컬럼 추가.

```sql
ALTER TABLE media_log
add (file_system_id int(11) unsigned);
```

- 파일 경로는 다른 로그도 같은걸 쓸 수 있기 때문에 굳이 외래키 제약조건을 달지 않는다.

```sql
ALTER TABLE `media_log`
	ADD INDEX `file_system_fk` (`file_system_id`);
```

- 인덱스는 달아준다.

#### 데이터 추가.

```sql
update media_log set file_system_id=floor(Rand()*((select count(1) from file_system)-1));
```
- 더미로 달아준다.

### 2. 테스트

- 원하는 테이블에 외래키 컬럼을 추가했으니 한번 보자.

#### 기본 query.

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null
)
select media_log.id, media_log.file_system_id, media_log.agent_name, file_full_paths.path from media_log join file_full_paths
on media_log.file_system_id = file_full_paths.id;
/* Affected rows: 0  찾은 행: 1  경고: 0  지속 시간 1 쿼리: 0.515 sec. */
```

- 아까 말했다시피 with recursive문은 temporal해서 그냥 select가안된다.
- 따라서 join할때마다 with recursive문은 추가해줘야한다.
- 귀찮으니 view로 만들자.

#### 뷰 추가.

```sql
create view file_path_view as (
    with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and fs.parent is not null)
    select * from file_full_paths
)
```

```sql
select media_log.id, media_log.file_system_id, media_log.agent_name, file_path_view.path from media_log join file_path_view
on media_log.file_system_id = file_path_view.id;
/* Affected rows: 0  찾은 행: 1  경고: 0  지속 시간 1 쿼리: 0.515 sec. */
```

![log-result]({{site.url}}/assets/img/query_full_path.png)

- 우선 log 데이터가 1개밖에없는데도 0.5초가 나온다...
- join 때매 그런걸까? log양에 linear하게 늘어날까?


#### 더미데이터 넣고 테스트

```sql
select a.id, a.file_system_id, a.agent_name, b.path from user_usb_log a join file_path_view b
on a.file_system_id = b.id;
/* Affected rows: 0  찾은 행: 117,350  경고: 0  지속 시간 1 쿼리: 0.547 sec. (+ 0.250 sec. network) */
```

- 11만행 넣고 테스트 해봤는데 차이가 별로 없다. 0.04초 차이.
- 신기한것은 알아서 recursive 한 system_id 대로 정렬이 된 상태로 나온다.?
- join을 하면 한 테이블의 clustered index 이자, 또다른 테이블의 nonclustered index인 외래키 컬럼으로 정렬 된 상태로 가져오면 훨씬 빠르기 때문인 듯 싶다.

![log-result]({{site.url}}/assets/img/query_full_path_many_log.png)

#### order by

- order by절로 자동 order를 풀어 보았다.

```sql
select a.id, a.file_system_id, a.agent_name, b.path from user_usb_log a join file_path_view b
on a.file_system_id = b.id
order by a.id;
/* Affected rows: 0  찾은 행: 117,350  경고: 0  지속 시간 1 쿼리: 1.093 sec. (+ 0.172 sec. network) */
```

- 훨씬 오래걸린다.

#### 더미데이터 변경

- 테스트 중에 확인한 한가지 사실은 기본적으로 recursive자체가 데이터 3만건임에도 불구하고 0.45초가 걸린다는 것 이었다.  <br/>
 중간에 더미데이터를 20000~에서 30000 row까지 무조껀 뎁스를 늘려가며 이전 데이터를 참조하도록 만들어서 그런 것 같았다.  <br/>
 tree구조로 보면 10000뎁스다.  <br/>
 테스트 데이터가 잘못 되었다는 얘기인데 이유는

1. 보통 path length를 줄이면 줄였지 늘리지는 않는다.
2. path가 4K 라는 것은 경로당 글자 1개, file Seperator 한개씩 2개로 쳐도 2K, 2000뎁스가 나온다.

- 따라서 데이터를 수정하기로 했다.

#### 10000

```sql
select * from file_path_view;
/* Affected rows: 0  찾은 행: 21,958  경고: 0  지속 시간 1 쿼리: 0.266 sec. (+ 0.015 sec. network) */
```

- 반으로 줄었다.
- 흠....
- row수가 줄어서 그런건지, 뎁스가 획기적으로 줄어서 그런건지 모르겠다.
- C드라이브 바로밑에 있는 폴더를 많이 만들어보자.


#### 25만개 row 2000뎁스

```sql
select * from file_path_view;
/* Affected rows: 0  찾은 행: 503,575  경고: 0  지속 시간 1 쿼리: 12.641 sec. (+ 0.109 sec. network) */
```

- 12초가 걸렸다.
- 근데 여기서 한가지 문제점을 발견했다.
- path에 fullText index를 넣었는데 파일 구조는 실재로 생각해보면 직전까지 같은 경로면, (즉 같은 부모를 갖고있으면) 이름이 unique하다.

#### 인덱스 재설정

```sql
ALTER TABLE `file_system`
	ADD UNIQUE INDEX `path_name` (`path`, `parent`) USING HASH;
```

- row 양은 어느정도 제한적이라고 생각해서 다중 컬럼 unique index를 해시 테이블로 만들었다. 

#### 50000개의 row, 10뎁스

```sql
select * from file_path_view;
/* Affected rows: 0  찾은 행: 49,974  경고: 0  지속 시간 1 쿼리: 0.891 sec. (+ 0.015 sec. network) */
```

- 뎁스의 양이 줄었는데 시간이 훨씬 늘었다.
- 단순 뎁스가 아닌 전체적인 양에 linear하게 시간이 늘어나는 것 같다.  

#### like 검색

```sql
select * from file_path_view where path like 'D://vw%';
/* Affected rows: 0  찾은 행: 5,008  경고: 0  지속 시간 1 쿼리: 0.907 sec. */
```

- 검색해서 행 수를 줄여도 마찬가지다.
- view 로 전체 recursive 한 CTE를 사용 해서 그런 것 같다.
- 앞전에 view로 CTE를 활용 한 후 where절로 활용 하는 방안을 버리고 recursive 사용 시 where절을 활용 하는 방안을 사용해보자.


#### recursive 튜닝

```sql
with recursive file_full_paths (id, path) as (
	select fs1.id, fs1.path from file_system fs1 where fs1.parent is null
		union
	SELECT fs.id as id,
		CONCAT(fp.path, '/', fs.path) as path
	FROM file_full_paths fp, file_system fs
	WHERE fs.parent = fp.id and 
		fs.parent is not null and
		fs.path like 'vw%'
)
select * from file_full_paths ;
/* Affected rows: 0  찾은 행: 48  경고: 0  지속 시간 1 쿼리: 0.031 sec. */
```

- like검색이 완벽히 되지는 않았지만 속도가 훨씬 빠르다.  
<br/>

##### 여기서 변명을 하자면

- with키워드는 recursive라는 것과 함께 CTE라는 이름으로 캐싱 최적화등의 고도화된 지원을 한다.
- 위에 with recursive subquery절은 CTE body라고 한다.
- 하단의 사용 하는 query 는 CTE Usage라고 한다.
- 개인적인 예상은 CTE Usage의 제한, 조건 절등을 보고 CTE body 가 알아서 잘 최적화 해줄 줄 알았다.
- 근데 상태를 보니 아니네..?  

<br/>

- 이제 여기서 문제점은
1. 여러 path에 걸친 검색이 안된다.
2. 부모 path검색하고나면 하위 path들이 검색이 안된다...


<br/>

###### 여기까지 하고 삽질을 마치겠다..

- 계속 파다보니 끝도 없고 너무 힘들다...
- 우선 마치고 방식을 바꾸기로 했다.

## 4. 마무리

1. 기본적으로 path는 4K를 최대로 맟추고 생각하면 된다.
2. mariadb varchar는 64K까지도 지원한다.
3. mariadb 는 한 row당 64K까지 지원하고 한 블록당 최대 64K도 지원한다.
4. 현재 filePath가 필요한 로그는 한 row당 1.6K정도 이다.
5. 외부 테이블을 설정하고 join하는거보다 column추가가 훨씬 이득이다.
6. 여기까지 오기까지 비용이 너무 많이 들었다.
7. 어차피 앞으로의 로드맵 중 로그 table은 빠르게 noSQL계열로 따로 db 파기로 되어있다.
8. jpa를 쓰고 있어서 db에서 처리된다고 하더라도 나중에 jpa로 옮겨오기까지가 많이 남아있다...

- 앞으로 방향성
  - varchar(4000) 으로 지정 *linux ntfs시스템이 default가 4K*
  - log에 column 추가. (로그성이고 join하면 훨씬 느려짐)
  - 추후 block size등으로 인한 성능저하가 있는지 주시하고 mysqld tuning 에 대한 고려.

쓰다보니 두서도 없고 의식과 공부의 흐름대로 쓴 포스팅이 되었는데 이제까지 포스팅중 가장 길다...
누가 이 포스팅을 읽을지 모르겠지만. 나한텐 큰 도움이 되었다.
CTE도 알게 되었고 마이크로한 DB성능 테스트 등 도 하게되었다.
나한테 좋은 공부가 되었다.
잊지 말자!.