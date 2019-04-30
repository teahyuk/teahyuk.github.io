---
layout: post
title: jvm에 대하여 - PermGen에 대해
feature-img: "assets/img/sample_feature_img.png"
categories : [JAVA]
tags : [JAVA, JVM]
---

#### 먼저 JVM 구조는 이렇다.
![jvm]({{site.url}}/assets/img/jvm1.png)

<br/>

#### 이중에 중요하게 알고 있어야 될 부분은 HEAP!
![heap]({{site.url}}/assets/img/jvm2.png)

 - ! 해당 이미지는 jdk 7 기준이다.

#### jdk 8 용 에서는 Perm Gen이 사라지고 metaSpace로 변경되었다.
 - Perm Gen에 각종 스태틱한 메모리들이 할당되고, 저장하는 기능이 있었는데 이부분을 metaSpace로 변경 함.
 - Perm Gen은 구동시 옵션 또는 default에 영향을 받은 static한 size
 - MetatSpace 는 permgen에서 쓰던 데이터들을 native 메모리로 저장 시키고 사이즈는 dynamic하다.(늘어난다.)
 - 일부 Perm Gen 에 저장하던 static 메모리들이 있었다. 이는 jdk7 부터 heap 영역으로 옮겨졌다. - 대표적으로 String...

- 출처
> <http://techidiocy.com/metaspace-java8/>
> <https://dzone.com/articles/java-8-permgen-metaspace>
> <http://openjdk.java.net/jeps/122>