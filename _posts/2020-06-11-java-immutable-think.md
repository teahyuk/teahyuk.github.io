---
layout: post
comments: true
title: JAVA VO, DTO 에 관한 고찰
feature-img: "assets/img/jeju.jpg"
categories : [Java]
tags : [Java, Functional, VO, DTO, Immutable, TDD]
---

## TDD 스터디 복습 중 불현듯 떠오른 고찰 Immutable 에 관하여

> 객체 설계 시에 사용자 로 하여금 알지 못하는 내부 상태 를 신경 쓰도록 설계 하지 말자

- 이 글의 결론 이자 스터디 복습 중 생각해 본 고찰 의 결과 이다.
- Vo, Dto 에 대해 좀 더 나만 의 기준이 생기게 되었다.

### 발단

Java study (TDD/Next step) 복습 중에 refactoring 하면서 자연 스럽게 객체 1개가 상태를 가지고 modify 메소드 와 getter 메소드 가 도출이 되길래, immutable 하게 어떻게 만들 수 있을 까 생각을 해 보았다.

- 다음은 Car 와 Racer 를 들고 있는 Player 라는 클래스 다.

```java
public class Player {
    //...
    private int distance;

    //...
    public void move(){
        if(car.movable()){
            distance++;
        }
    }

    public int getDistance() {
        return distance;
    }
    //...
}
```

- 곰곰히 생각해 보면 어차피 distance 는 말단 사용자 (UI) 단에서 매번 확인 하고 들고 있게 된다.
- 뭔가 해당 Player 객체 들이 Thread Safe 하면서 도 함수형 으로도 쉽게 접근 할 수 있도록 immutable 한 객체로 만들고 싶은 욕구가 생겼 는데 어떻게 해야 되는가!!???
- 가변 변수인 distance 라는 상태 값 부터 없애야 한다!?

### 전개

- 요구 사항 부터 다시 확인 해보자!
    - 해당 게임은 매 턴(turn? round?) 마다 현재 상태를 UI 에 그려 줘야 한다.
    - Player 가 현재 distance 를 꽁꽁 싸매고 들고 있으면 서 나중에 보여주 거나 할 이유가 없다.?
- 그렇 다면 그냥 현재 distance 를 받아서 다음 distance 만 던져 주면 되지 않을까?

```java
public class Player {
    //...
    public int nextRound(int currentDistance) {
        if(car.movable()){
            return currentDistance+1;
        }
        return currentDistance;
    }
    //...
}
```

- 우선 1단계 이 정도 로 된 것 같다.
- 매번 현재 단계의 주행 거리를 받게 했고, 다음 단계의 주행 거리를 리턴 하도록 했다.
- 이로 써 '주행 거리' 라는 상태를 Player 가 갖고 있지 않아도 되게 되면서 immutable 이 되었다.

```java
public class Player {
    //...
    public Distance nextRound(Distance distance) {
        if(car.movable()){
            return distance.add();
        }
        return distance;
    }
    //...
}
```

- int distance 를 VO 로 만들어 사용 할 수 있게 바꿨다.
- 이 정도 면 해당 Player 객체를 사용 하는 사용자 객체 가 헷갈리 지 않게, Thread 위험 성에 침해 되지 않게 설계가 잘 된 것 같다.
- 이제 이 Player 객체 들을 들고서 Racing Game 을 수행 하는 객체 가 필요 할 것 이다. 

### 결론

위의 refactoring 과정 에서 변경 된 부분은 다음 과 같다.

1. 가변 객체 에서 상태 값을 뺏다.
2. 상태 의 변화 는 객체 의 내부 상태 값이 아닌 argument 로 받은 상태 값 과 그에 맞는 리턴 으로 대체 했다.
3. 상태 로 주고 받는 데이터 는 primitive 타입 이 아닌 상태 객체 (VO) 로 만들어 사용자 가 알기 쉽게 명시 했다. (그 와 동시에 add() 로직 도 내부로 넣었다)
 
이로 써 얻을 수 있는 이점 은 
1. Player 객체는 ThreadSafe 하므로 사용자 객체가 신경 쓸 필요가 없다.
2. Distance 라는 VO 로 input output 을 주고 받기 때문에 해당 값이 무엇 인지 알 기 쉽다.

### 고찰

이 과정을 겪으면 서 VO, 또는 DTO 에 관한 고찰이 생겼다. 

- VO 는 Value Object 로 써 해당 객체 자체로 '값' 즉 '상태' 를 나타 내는 객체 이다.
- DTO 는 Data Transport Object 로 써 기본 개념인 통신, 또는 메세지 교환 간에 Data 정보를 저장 하기 위한 객체 이다.

위의 VO 와 DTO 의 정의 를 곱 씹어 보면 `값`, `상태` 를 나타 내는 객체 인 것 이다.

여기서 또 한가지 생각 할 수 있는 것이

- java 에서 상호 객체 간의 method `호출` 과 `응답` 은 하나의 `통신`, `교환` 이라고 볼 수 있다.
  - 여담 이지만 그래서 rmi 같은 게 나온 거 아닐 까?
  - C# 에서는 method 호출을 아예 메세지 통신 의 일환 으로 본다.
- 그렇 다면 method 호출 에서 argument 는 통신 을 위한 Data 정도 로 생각 할 수 있다.

VO, DTO 가 나오게 된 원인 과, 그에 관한 통찰이 개발 5년차 인 이제서 생겼다....

~~그동안 여러 레거시 코드를 보면서 힘들었 던 수많은 기억 들이.. 해당 코드를 고칠 만한 방향 성이 이제 서야..ㅠㅠ~~

Service 나 Controller, 또는 각자 의 객체 등 등 에서 business 로직 호출 시에 '상태' 라는 놈은 객체 **내부에 숨기는 것이 아니라** '상태' 를 나타 내는 또 다른 '객체'(VO,DTO) 를 만들어 서 해당 객체로 통신 해야 한다.

주저리 주저리 썼지만 결론은, 객체 설계에 대한 기본 적인 내 태도는 정해 졌다.

> 객체 설계 시에 사용자 로 하여금 알지 못하는 내부 상태 를 신경 쓰도록 설계 하지 말자

적어도 java, oop 에 관한 개발 시에는 이런 생각을 잊지 말고 져 버리지 말고 개발 해야 할 것이다.
