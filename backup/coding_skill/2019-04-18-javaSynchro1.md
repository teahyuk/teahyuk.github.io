---
comments : true
title: java에관한 잡설 - synchronize
feature-img: "assets/img/sample_feature_img.png"
categories : [JAVA]
tags : [JAVA, Synchronize, Syntex]
---

> java synchronize 에대한 개인적인 고찰입니다.

<br/>
<br/>

#### 오늘은 ConcurrentHashMap의 동작에 대해서 개인적으로 파다가 synchronize 에 기본 동작에 대해 정리한다.
<br/>

1. synchronize 는 C#에서는 lock과 같음. 
2. 메소드에 거는 synchronize 는 해당 메소드를 가진 인스턴스를 기준으로 lock이 걸림.
    - 메소드 여러군데 synchronize걸면 다같이 lock걸림
    - new해서 새로운 인스턴스가되면 같은메소드라도 lock안걸림
3. synchronize 라는 것은 java의 모든 객체(레퍼런스객체)에 존재하는 기본 락 - (instrinsicLock이라 함) 에다가 거는거임
4. instrinsicLock은 기본적으로 reentranceLock - (이미 락 획득중인 쓰레드)면 새로진입 계속 되는 것.
5. instrinsicLock은 레퍼런스 타입에만 존재
    - primitive타입에 안됨.
    - Boxing클래스들은 기본적으로 immutable클래스라 Operation 이후에는 새 인스턴스가 생성됨 (String도 마찬가지 - Not StringBuilder)
    - 따라서 새 인스턴스 생성되므로 Synchronize 동작 안함.

EX)
~~~java
Integer x = 0;
x++;
equals
x = new Integer(x+1);
~~~    