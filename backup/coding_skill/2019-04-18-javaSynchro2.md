---
comments : true
title: java에 관한 잡설 - lockSplit vs lockStriping
feature-img: "assets/img/sample_feature_img.png"
categories : [JAVA]
tags : [JAVA, Synchronize, ConcurrentHashmap]
---

> java synchronize 핸들링에 관한 정의 입니다.

<br/>
<br/>

## ConcurrentHashMap의 동작에 관해 개인적으로 파다가 lockStriping에 관해 정리한다.
<br/>

### 1. ConcurrentHashMap은 기본적으로 lockStriping을 사용한다.

<br/>

### 2. synchronize 핸들링방법은 3가지
1. 무식하게 메소드시그니쳐에 synchronize => synchronize(this)
1. lockSplit 
1. lockStriping

<br/>

### 무식한건 건너뛰고오..

<br/>

### 3. lockSplit
- lockSplit은 클래스 멤버변수 기준 sync나누는 기법

~~~java
class SplitedConcurrentClass{
    private int atomicCount;
    private Object atomicVal;
    private final Object lockCntObj = new Object();
    private final Object lockValObj = new Object();

    public void setCnt(int cnt){
        synchronize(lockCntObj){
            ...
        }
    }

    public void setVal(Object val){
        synchronize(lockValObj){
            ...
        }
    }
}
~~~

<br/>

### 4. lockStriping
- lockStriping은 클래스중심이 아니라 메모리 접근 중심으로 생각해서 세그먼트단위 나누고 구역별로 sync하는 방식
- concurrentHashmap이 이렇게 씀
- jdk7때는 16개의 세그먼트로 나누고 쓰곤 했지만 jdk8때는 각 HashKey마다 한개 세그먼트씩 할당해서 sync하게 함.
    - 이는 해시충돌 일어나서 이후 아이템 확인 할 때만 sync로 보장하고 나머지는 접근조차 달라서 따로 sync둬도 됨.
    - 굳이 다른 해시키를 세그먼트단위로 같은 sync안에 묶을 이유 없음.

예시) array가 필드에 할당되면 하나로 묶는게아니라 내부 메모리 접근정도 각각 확인해서 분리해서 sync하게 함. => HashMap이나 LinkedList 같은것들 쓰더라도 마찬가지 내부 자료구조 파악한 뒤에 메모리 접근 위치 다르면 각각 다른 obj할당해서 sync걸음.
~~~java
class StripedConcurrentClass{
    private int atomicCount;
    private Object[] atomicVal;
    private final Object lockCntObj = new Object();
    private final Object[] lockValObj = new Object();

    public void setCnt(int cnt){
        synchronize(lockCntObj){
            ...
            lockValObj::add(cnt);  //수도코드임 걍 cnt따라서 올리도록하는거.
        }
    }

    public void setVal(Object val,int idx){
        synchronize(lockValObj[idx]){
            ...
            atomicVal[idx]::blabla..~~    //atomicVal[idx]만 만지도록 함.
        }
    }
}
~~~