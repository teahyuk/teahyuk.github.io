---
layout: post
comments: true
title: 70번
feature-img: "assets/img/jeju.jpg"
categories : [EffectiveJava]
tags : [effectiveJava, JAVA]
---

## 복구 할 수 있는 예외는 검사 예외를 복구 불가능한 예외는 비검사 예외(RuntimeException | Error)를 사용하라

>
호출자에서 그대로 복구해서 사용 가능할 꺼같으면 검사 예외다 (throws, Extends Exception &!(RuntimeException | Error))

### throws로 검사 예외를 만들어내는 경우는 api호출자가 복구 가능한 경우에만 만들자

```java
public int[] getIntArray(int count) throws OutOfMemoryError{
    int[] a = new int[count];
    return a;
}
```

- OOME 이딴 말도 안되는 코드는 나올 수도 없고 않쓰는게 맞다.
- 당연한 얘기지만 이미 메모리 overFlow난 상황에서 시스템이 더 동작 하는게 불가능하다.
- 걍 뒤지는게 답. => 재시작.. ㄱㄱㄱ

### throwable만 상속하여 만든 커스텀 예외 금지!

- 이건 몰랐던 사실인데 throwable만 상속해서 커스텀 예외를 만들 수 있다.?
- 근데 java기본 개념을 깨트리는 말도안되는 구조이므로 하지말자.
- 무조건! Exception, 또는 RuntimeException만. 상속!
- Error는 쓸일이 없다. 컨트롤 할 일도 없고.

### 예외 역시 완벽한 객체이다

- Exception은 주로 그 예외를 일으킨 상황에 관한 정보를 코드 형태로 전달하는데 쓰인다.
  - 다시말하자면 Exception 클래스 명 자체에 정보가 있다. (Exception명 잘 만들자)

- 예외를 전달하기 위해 별도의 String을 파싱하는 모양을 만들지 말자.

### 검사 예외 처리 시에는 해당 예외를 던지는 객체에서 예외 해결법에 대한 메소드도 제공해야한다

- 이거 중요하다.
- 예를들어 현재 next와 goBack등의 현재 포인터 이동만 제공하는 iterator 객체? 가 있다고 치자.
  - IndexOutBoundException을 던지게 된다면 해결 법도 나와야한다.
  - 즉 예외 해결을 위해 현재 포인터의 위치도 알 수 있을 법한 position 등의 메소드도 제공해야한다.
