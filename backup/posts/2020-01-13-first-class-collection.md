---
comments: true
title: 자바 일급 컬렉션
feature-img: "assets/img/jeju.jpg"
categories : [JAVA]
tags : [JAVA, Collection]
---

## 이동욱 님 블로그 발췌

> https://jojoldu.tistory.com/412?category=635881

java에서 Map 또는 List등의 Collection사용 시에 일급 컬렉션을 사용하는것이 신상에 이롭다는 클린 코드 운동 방식 중 한가지이다.

이동욱(jojoldu)님 블로그를 보고 동의를 하는 내용이며 자꾸 까먹기 때문에 정리하며 적어둔다.

## 일급 컬렉션이란.

특정 자료구조 클래스 (VO Class) 중 멤버변수가 List, Map 등등 자바 Collection Util class 한가지밖에 없는 자료구조를 얘기하는데 자바 기본 api만을 사용해서 처리하는 것 보다 일급 컬렉션을 만들어서 사용 하는것이 깨끗한 코드 관리법에 좋다 하여 생긴 방법이다.

```java
public class EventData {
    private Map<String, Object> eventLog;

    public EventData(Map<String, Object> eventLog) {
        this.eventLog = eventLog;
    }

    public Object getValue(String key){
        return eventLog.get(key);
    }
}
```

### 1 자료구조의 네이밍 및 가시성 증가

- 당연한 것이 그냥 List, Map 이런 모양으로 작업하면 변수명을 잘 써야 하는 이슈가 생긴다.
- 그래서 클래스를 따로 만들어서 비즈니스에 어울리는 도메인으로 네이밍 한 자료구조로 만들면 작업 하기가 쉬워진다.

### 2 immutable

- 요즘 java 에서 동시성 함수형 등등의 요구사항으로 immutable 이 중요해지고 있다.
- 나도 immutable 의 장점을 잘 알고 있으며 좋아하는 입장에서 확실한 immutable이 중요한 가운데 저거는 좋은 장점인 것 같다.
- 그냥 List, Map등등 쓰면 내부 변수 set에 대한 접근 제한이 되지 않는다. 따라서 일급 컬렉션으로 만든 뒤 set관련 함수는 제거하여 immutable하게 만들 수 있다.

### 3 비즈니스 로직 관리

- 간단히 얘기해서 알맞는 관심사로의 비즈니스 로직을 변경 할 수 있다.
- 예를들어 특정 도메인에서의 특성으로 분리 될 수 있을 만한 로직 들은 아무 일도 하지 않는 기본 컬렉션을 이용하면 해당 자료구조 사용자 코드에 지속적으로 복붙하여 중복 작업을 하도록 나타나게 될 수 밖에 없다.
- 이러면 Util 클래스를 만들고 싶은 욕구가 생기게 되기도 한다.
- 도메인 상태나 도메인 특성 자체에 대한 로직등이 생기면 일급 컬렉션으로 몰아넣고 사용하면 좋다.

#### 다음은 위에 예제에서의 EventData는 Schemaless한 자료구조이지만 기본적인 ID 만은 UID라는 이름으로 정해져 있다는 validation을 갖고 있다는 것을 해결하기 위한 코드다.

``` java
public class EventData {
    private String ID_KEY = "UID";

    private Map<String, Object> eventLog;

    public EventData(Map<String, Object> eventLog) {
        if(!eventLog.containsKey(ID_KEY)){
            throw new IllegalArgumentException("eventLog must have UID Key");
        }
        this.eventLog = new HashMap<>(eventLog);
    }

    public String getId(){
        return (String)eventLog.get(ID_KEY);
    }

    public Object getValue(String key){
        return eventLog.get(key);
    }
}
```

#### 이렇게 했을 때 모든 EventData를 사용중인 코드들이 UID에대한 Validation Check를 하지 않아도 된다.