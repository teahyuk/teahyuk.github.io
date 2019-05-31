---
layout: post
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

윈도우즈는 2016년 win10에서 그동안 제한 걸려있떤 260 길이의 경로 제한을 풀어버렸다.
> https://mspoweruser.com/ntfs-260-character-windows-10/

<br/>
<br/>

##