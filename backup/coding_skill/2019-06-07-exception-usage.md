---
comments: true
title: 69번
feature-img: "assets/img/jeju.jpg"
categories : [EffectiveJava]
tags : [effectiveJava, JAVA]
---

## 예외는 진짜 예외일 때만 사용해라

> 예외를 조건등 다른 걸로 사용 하지 말자.

### 예외를 사용해서 로직을 수행하는 경우는 만들면 안된다

```java
List<Intager> a = new ArrayList<Intager>();
try{
    while(true)
        int tmp = a.next();

} catch(ArrayOutBoundException e){
    system.out.println("array is done");
}
```

- 아주 간단한 쓰래기같은 예제이다.  
- 할말 없다.. 1년차만 지나도 이런건 안하게된다.  
- 요즘 추세는 오히려 함수형이 발전되면서 발생하던 Exception들도 내부에서 왠만하면 처리하여 외부에서 복구시키는 로직을 수행하도록 장려하는 추세이다.
  - Stream에서 Exception을 바깥을 못꺼낸다든지.
  - Rxjava2에서 onError 의 구독자는 따로 있게 되어있다든지.

#### 어찌됐든!!

- 하지말자.
