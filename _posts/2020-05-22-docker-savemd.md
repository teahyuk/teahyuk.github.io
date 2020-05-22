---
layout: post
comments: true
title: Docker 이미지 파일 save load
feature-img: "assets/img/jeju.jpg"
categories : [Services]
tags : [Docker]
---

## Docker 이미지 폐쇄망 이동을 위한 tar save 및 load 방법

> Docker를 사용하다보면 폐쇄망 또는 registry없이 이동 하기 위해 파일로 이동을 해야 할 때가 필요하다

### Docker Save

```sh
sudo docker save image:tag > image.tar
```

```sh
sudo docker save image:tag -o image.tar
```

이렇게하면 tar파일로 저장이 되고, 해당 파일을 옮기고자 하는 노드로 넘겨서 load하면 된다.

### Docker load

```sh
sudo docker load < image.tar
```

```sh
sudo docker load -i image.tar
```

#### 주의

해당 옵션은 별것이 없고 이미지의 이름과 태그가 전부 다 따라와서 load 되기 때문에 이미지 이름을 변경 하고 싶다면 따로 이미지 이름 변경을 실행 해야 한다.
