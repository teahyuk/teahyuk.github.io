---
layout: post
comments : true
title: singleton 정리
feature-img: "assets/img/sample_feature_img.png"
categories : [JAVA]
tags : [JAVA, Singleton]
---

> 요즘 자바로는 간단하게 싱글톤으로 구현시켜주는 Enum 클래스가 존재한다.

싱글톤에 대하여 가장 최근에 들어 본 것은 EffectiveJava 3판에 나온 Enum을 활용한 방법이다.

자바에서 기본 적으로 지원해주는 클래스인 Enum클래스의 특성 덕분인데, 해당 클래스의 모든 인스턴스는 Static하게 사용 하고, new는 private으로 함부로 new 를 쓸 수 없으며, 하나의 인스턴스만 공유해서 해주기 때문에 (원래는 Enum을 지원하기 위해 이렇게 우회적으로 만들어졌다.)
Enum을 클래스처럼 활용하고 하나의 Enum값(Static한 인스턴스) 를 만들면 싱글턴 처럼 사용 가능해진다.

현업에서는 보통 DI툴을 활용해서 인스턴스 공유 방식으로 싱글톤을 대체하기 때문에 잘 안쓰지만. 토이프로젝트에서 써보고 괜찮으면 적극 활욜 해보려 한다.
<br/>
<br/>
<br/>

원래는 싱글톤 전체 구현에 대한 글을 기록하려 했는데 쓰다보니 Enum 싱글톤에 대한 찬양이 되어버렸네..
해서 아래부터는 싱글톤들의 발전 과정과 역사다.



### 1. 기본형
- 초기화 할 때 무조건 객체 할당 하게되고, 이게 싱글턴 클래스끼리 생명주기 에 관해 컨트롤 못해서 순서 없이 동작하도록 짜거나 해야함.
- 누군가 쓰지 않더라도 강제 할당.

```java
public class Singleton{
    private Singleton instance = new Singleton();
    
    private Singleton(){}

    public static Singleton getInstance(){
        return instance;
   }

   ...
}
```

### 2. lazy 초기화 그리고 쓰레드세이프를 위한 synch
- 성능이 개 후짐.
```java
public class Singleton{
    private Singleton instance;
    
    private Singleton(){}

    public static synchronized Singleton getInstance(){
        if(instance == null)
            instance = new Singleton();
        return instance;
   }

   ...
}
```

### 3. lazy + 성능 완화를 위한 double 체크 lock
- 생성에 관한 locking만 하기때문에 instantiate과정이 길면 다른 쓰레드가 와서 참조하고 쓸 때 문제가 생길 수 있다. 미완성.

```java
public class Singleton{
    private Singleton instance;
    
    private Singleton(){}

    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
   }

   ...
}
```

### 4. Enum
- 맨처음에 언급해서 엄청 찬양을 하였지만... 결국에 lazy init을 못한다는 얘기는 1번과 같은 문제점이 있을 수 있다는 것.
- 무조건 다른 singleton 클래스와 연관성이 없이 짜야 한다는 것... 또는 초기화 되지 않은 다른 객체를 참조하는 작업 등등이 있을 때 조심해서 써야 한다는것...?

```java
public enum Singleton{
    INSTANCE;

    ...
}
```

### 5. layzHolder
- 아직까지는 이게 완전체
- 이 마저도 reflection을 이용한 java class해킹? 방법으로 악의적인 사용자가 singleton을 풀어서 사용 가능

1. private inner class는 해당클래스를 감싸고있는 외부 클래스를 참조해서 들어가지 않는 이상 클래스초기화 => static초기화를 하지 않는다.
2. 따라서 Singleton.getInstance() 시에 lazyHolder 클래스가 로딩된다.
3. lazy하게 singleton 할당 가능하다.
4. static이라서 쓰레드에도 안전하다.

```java
public class Singleton{
    private Singleton(){}

    private static class lazyHolder{
        private static final Singleton Instance = new Singleton();
    }

    public static Singleton getInstance(){
        return lazyHolder.Instance;
    }

   ...
}
```

### 부록
- 위에 5번 lazyholder 에서 설명 했듯이 reflection과 serialization, clone등을 이용한 공격? 에도 대항하고 막는 등의 치열한 공방의 과정들이 인터넷 여기저기에 널려있다.
- 호불호가 갈리지만 흥미로운사람들도 있을 수 있으니 확인해보자.
    - > 내가 좋아하는 Enum도 깰 수 있단다 ㅠㅠ 참고 : https://www.javacodegeeks.com/2013/06/singleton-design-pattern-a-lions-eye-view.html


### 결론
1. 이러나 저러나 아직 lazyholder만한 놈이 없는 것 같다.
2. 쓰고 정리하면서 드는 생각인데 Di로 인스턴스 공유해서 쓰는게 차라리 낫다는 생각이 든다.
    - new 는 상속이 안되니깐, 상속해서 뭔가사용 하는거 못한다.
    - 테스트 코드 짤 때 저 singleton이 방해가 된다. -> biz로직에 singleton 쓰는 구간 분리해서 쓸 때 상속도 안되니 mock객체같은걸로 갈아끼지도 못한다.
    - 진짜 쓸만한 곳만 잘 쓰는게 나을듯....