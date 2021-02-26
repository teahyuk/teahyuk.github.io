---
comments: true
title: java Mockito VS PowerMock
feature-img: "assets/img/jeju.jpg"
categories : [IdeaLog]
tags : [JAVA, JUnit, powerMock, Mockito]
---

## 우선 결론!

> **Mockito보다는 PowerMock 쓰는 것을 추천한다!. 아니 반드시 저거쓰자..**

Junit을 사용하여 새롭게 나타난 이슈에 대해 테스트를 해야 할 일이 생겼다.
원인은 서로다른 api간의 타이밍으로 인해 나타난 데이터 정합성 이슈였고 해당 api가 제 3자 서비스까지 접근하여 값을 읽어오는 바람에 네트워크 응답 타이밍에 대한 테스트가 필요한 상황이 되었었다.

## 문제 api 동작과정

1. 클라이언트한테 api 호출을 요청 받으면 서버는 빠르게 3자 서비스에 이미 완료된 요청이 있는지 확인 한 후 현재의 서버로 상태를 갱신하고 변경한다
2. 클라이언트가 3자 요청까지 기다릴 순 없으므로 자신의 상태를 변경하기위해 해당 DB에 수정요청을 따로 가한다
3. 모든 api호출 및 네트워크 구간은 비동기이므로 비즈니스상 순서대로 동작해야 할 동작이 꼬이면(3자 서비스보다 클라이언트 수정요청이 한번더 먼저들어오면) 3자서비스에서 리턴 받은 초기 데이터상태로 다시 초기화 하게 된다

> 지금부터 비즈니스 로직 에 대한 이해는 불필요 하므로 지금 포스팅의 중점인 Mockito 삽질 과정만 정리할 것이다!!

## 최초 Mockito 사용 과정

1. 간단하게 제 3자서비스로 요청하는 HttpClient Send부분만 mocking하면 될 것이라 생각했다
2. mocking된 클래스에 테스트용 임시 응답리턴만 남기고, latch로 지연 시키려 했다
3. 코드 *(참고로 실제 업무에 사용된 코드를 비슷하게만 따라 쓴 것이며 순수 java이지만 이해가 쉽게 Spring비스무리하게 이름 변경했다)*

```java
//import 및 초기화등등 생략

public class AnalysisTest{
    @Test
    public void AnalysisTimingTest(){
        //대기 래치
        CountDownLatch countDownLatch = new CountDownLatch(1);
        //대기를 위한 목킹
        Core3rdHttpService mockCoreHttpService = spy(new Core3rdHttpService());
        when(mockCoreHttpService.requestData(anyString(),any(),any()))
            .thenAnswer(invocation->{
                countDownLatch.await();
                return new HttpEntity("응답메세지");
            });
        //목킹 http사용하는 api 생성
        Use3rdServiceController use3rdServiceController = new Use3rdServiceController(mockCoreHttpService);
        //클라이언트에서의 요청
        Future<HttpEntity> recievedFuture1 = use3rdServiceController.request1("요청1");
        //다른요청 추가.
        Future<HttpEntity> recievedFuture2 = use3rdServiceController.request2("요청2");
        //대기래치 실행
        countDownLatch.countDown();
        //요청1확인
        assert(recievedFuture1);
        //요청2확인
        assert(recievedFuture2);
    }
}
```

## Mockito의 빈약함! 그리고 NPE

근데 여기서 테스트 시작도 못하고 NPE가 나타났다

원인은 위에 spy로 mocking한 Core3rdHttpService 가 문제였다.

```java
public class Core3rdHttpService{
    public Future<HttpEntity> requestData(String dataId, @NonNull EnumObject enumType, @NonNull DataObject analysisdata){
        return httpClient.send(dataId,enumType.getJsonValue(), analysisData.getRawData());
    }
}
```

위의 코드에 보면 ```when({mockMethod})``` 부분에서 실제 when으로 method응답을 가로채기전에 미리 한번 실제 객체 메소드에 요청해본다 *(when이 먼저 나오고 무엇으로 리턴할지 몰라 미리 argument로 methoCall을 해보기 떄문)*

그런데 안에 argumentMatcher로 들어가는 부분 코드들을 보아하니,

```java
public class Matchers {
    private static final MockingProgress MOCKING_PROGRESS = new ThreadSafeMockingProgress();

    public Matchers() {
    }

    public static <T> T anyInt() {
        return reportMatcher(Integer.Int).returnZero();
    }

    public static <T> T anyObject() {
        return reportMatcher(Any.ANY).returnNull();
    }

    public static <T> T any() {
        return this.anyObject();
    }

    public static <T> T any(Class<T> clazz) {
        return reportMatcher(clazz).returnNull();
    }

    public static <T> T any(T instance) {
        return reportMatcher(instance).returnNull();
    }

    //... 완전히 모든 any 또는 eq등등이 다 return은 기본값으로 함.
```

맞다, 싹다 기본값이다 null,0,false,'\u0000',new array[0], new List() 등등...

게다가 내가 모킹해서 자동 응답처리하려는 3rdParty의 모킹 서비스는 enum형을 받는다. *(enum도 instance라 내부기능상은 숫자지만 java에선 null로가능...)*

## PowerMock으로의 전환

mockito를 사용하지않고 자체 mocking프록시 클래스를 만들까, 저 Matcher를 수정해서 사용해볼까 여러가지 고민을하면서 수 시간을 보내다가 powerMock을 발견했다.

이미 많은 사람들이 mockito가 빈약하여 powerMock을 사용하던 중이었고, 또 다른 여러가지 openSource들을 사용하는 듯 보였다.

나는 뭐 오래 됐을수도 있지만 그래도 사람들이 제일 많이 사용하는거같이 보이는걸 선택했다.

- https://www.javadoc.io/doc/org.powermock/powermock-api-mockito/1.7.0/index.html

```java
//powerMock 기본사용법
doReturn(returnObject)
    .when(mockInstance)
    .mockMethod(argumatcher1(),argumatcher2()...);
```

1. 행위를 먼저 기술하고, 
2. mockingInstance만으로 조건을 감싸며, 
3. mocking할 Method를 최후에 지정함으로써 
4. 쓸데없이 mocking을 하기위해 기존 method를 미리 실행 시키거나 하지 않는다.

후에 HttpClient도 Netty를 사용하느라 3rdPartyService에서도 NettyFuture를 리턴하는것을 알고 NettyPromise를 이용해 Latch도 제거하였다.

## 최종 코드

```java
//import 및 초기화등등 생략

public class AnalysisTest{
    @Test
    public void AnalysisTimingTest(){
        //대기를 위한 목킹
        Promise<HttpEntity> returnMockedMethod = new DefaultPromise<>(ImmediateEventExecutor.INSTANCE);
        Core3rdHttpService mockCoreHttpService = spy(new Core3rdHttpService());
        doReturn(returnMockedMethod)
            .when(mockCoreHttpService)
            requestData(anyString(),any(),any());

        //목킹 http사용하는 api 생성
        Use3rdServiceController use3rdServiceController = new Use3rdServiceController(mockCoreHttpService);
        //클라이언트에서의 요청
        Future<HttpEntity> recievedFuture1 = use3rdServiceController.request1("요청1");
        //다른요청 추가.
        Future<HttpEntity> recievedFuture2 = use3rdServiceController.request2("요청2");
        //3rdParty응답
        returnMockedMethod.setSuccess(new HttpEntity("응답메세지"));
        //요청1확인
        assert(recievedFuture1);
        //요청2확인
        assert(recievedFuture2);
    }
}
```