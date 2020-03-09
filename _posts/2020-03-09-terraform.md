---
layout: post
comments: true
title: TerraForm
feature-img: "assets/img/jeju.jpg"
categories : [Services]
tags : [Infra,IaC,Terraform]
---

인프라를 위한 코드 구축 툴로써, java에서의 maven과 같은 역할을 해준다고 생각하면 된다.

단 코드레벨이 아닌 인프라레벨 이다

- ex) docker 이미지 생성 push, k8s 클러스터 관리, aws 인스턴스 생성 등등

> 클라우드 구축 시, 또는 각종 인프라 구축시 필요한 작업들을 도와주는 <span style="color:#e11d21">**IaC**</span> 툴 [wiki](https://ko.wikipedia.org/wiki/코드로서의_인프라스트럭처)

## 기본 사용 법

www.terraform.io 에서 다운로드 후 cli로 사용.

기본적으로 하시코프 언어라는 선언헌 명령어로 구성 된 .tf 파일을 만들고, 그 안에서 다운 받은 terraform 파일 실행시켜 cli로 사용 하면 된다.

### 기본 명령어

1. terraform init
    - 테라폼 언어 파일의 선언문 들을 보고 기본적으로 필요한 plugin들을 세팅하고 설정하는 동작 과정 (provider가 바뀌거나 호스트 os가 바뀌면 동작시켜야 하고, <https://terraform.io> 과 연결 되어있어야 한다.)
2. terraform plan
    - .tf 파일 추가 또는 수정 후 terraform 명령어 실행 시 tf파일 변경 사항을 체크하거나 validation 을 체크한다. (로컬 테스트에서 돌려보면 좋다)
3. terraform apply
    - .tf 파일을 최종 실행한다. (이 동작으로 인프라가 반영된다)

## HCL

하시코프 언어(HashiCorp Language)로써 terraform 실행을 위한 선언적 언어이다.

terraform 실행 시에는 .tf 파일로 저장한다.

### provider

필요한 각종 인프라 도구들에 대한 선언이다.

- ex) aws, docker, k8s, ansible, etc....

참고로 각 Provider 마다 모두 사용하는 언어 구성이 다르다.

- 지원 Provider 리스트 <https://www.terraform.io/docs/providers/index.html>

> 이후 아래 모든 예제들은 docker provider 기준으로만 작성한다.

```sh
provider "docker" {
  host="unix:///var/run/docker.sock"

  registry_auth {
    address = "docker-hub.private-host.com"
    config_file = "/root/.docker/config.json"
  }
}
```

### resource

provider 에서 제공 해주는 생성, 사용 가능한 리소스들 이다.

문법은 resource "{Provider_name}_{Resource_name}"이다.

```sh
resource "docker_container" "foo" {
  image = "${docker_image.centos.latest}"
  name  = "terraform-centos-latest"
}

resource "docker_image" "centos"{
  name = "docker-hub.private-host.com/centos"
}
```

이 것도 각 provider마다 다르므로 terrform doc에 잘 나와 있으며 다음을 참조하면 된다.

- ex) docker provider 의 컨테이너 리소스 사용법 <https://www.terraform.io/docs/providers/docker/r/container.html>

### data

data 는 provider 가 제공해주는 읽기 전용 정보를 가져오는 내용이다.

```sh
data "docker_registry_image" "ubuntu" {
  name = "ubuntu:precise"
}
```

해당 방식도 provider마다 제공해주는게 다르므로 docu를 살펴보자

- ex) docker provider의 docker_registry_image data 사용법 <https://www.terraform.io/docs/providers/docker/d/registry_image.html>

### variable

매개 변수 또는 공통 변수를 지정하는 문법이다.

terraform apply 사용 시 옵션값을 받을 수도 있고, 기본 값을 지정해 공유하여 사용 할 수도 있다.

```sh
variable "centos-image-name" {
  description ="centos의 도커 이미지 이름"
  default="centos"
}
```

### output

terraform plan 또는 apply 시에 콘솔로 출력 하는 내용에 대한 선언문이다.

```sh
output "centos-container-tag-name" {
  value = "${docker-image.centos.latest.name}"
}
```

## 오프라인에서 활용 법

신규 플랫폼은 private cloud 환경에서도 동작이 되어야 한다.

이는 인터넷과 망 분리 된 상태일 수 있기 때문에 개발 시에 오프라인으로 동작 할 수 있게끔 구축을 해놓고 사용 해야 한다.

### terraform-bundle

위에 설명한 부분과 마찬가지로 init 할 때에만 terraform.io 로 접근을 수행한다.

따라서 init 수행 시에 필요한 각종 plugin들을 미리 함께 지정해둔 terraform-bundle을 패키징 한 후에 배포하고 init을 수행하면 terraform.io로 접근하지 않으므로 오프라인에서 수행이 가능하다.

### terraform-bundle 다운 및 설치

플랫폼 최초 개발시 온라인 환경에서 필요하며 github에 terraform프로젝트를 다운 받으면 ./tools/terraform-bundle 이 있다.

- <https://github.com/hashicorp/terraform/tree/master/tools/terraform-bundle>

terraform은 go 언어로 이루어져 있기 때문에 terraform-bundle 이라는 툴 사용시에 go언어로 compile 및 install을 해야한다.

따라서 적정한 go 언어부터 설치한다.

- <https://golang.org/dl/>

go 언어로 install 하게 되면

```sh
go install ./tools/terraform-bundle
```

```%GOPATH/bin%``` 에 terraform-bundle이 나온다고 되어있는데 ```%GOPATH%```는 default 로 보통 ```$HOME/go``` 이지만 환경에 따라 다를 수 있다.

이제 terraform-bundle을 사용 할 수 있다.

### bundled-package 만들기

편하게 모든 plugin들을 미리 다 갖고 있는 package를 만들어도 되고 사용 할 인프라들의 provider plugin들만 갖고 있는 package를 만들어도 된다.

똑같이 hcl 언어를 활용하여 packaging 선언 파일을 만든다.

- terraform-bundle.hcl 파일
  
    ```sh
    terraform {
      # Version of Terraform to include in the bundle. An exact version number
      # is required.
      version = "0.12.21"
    }

    # Define which provider plugins are to be included
    providers {
      # Include the newest "aws" provider version in the 1.0 series.
      aws = ["~> 1.0"]

      # Include both the newest 1.0 and 2.0 versions of the "google" provider.
      # Each item in these lists allows a distinct version to be added. If the
      # two expressions match different versions then _both_ are included in
      # the bundle archive.
      google = ["~> 1.0", "~> 2.0"]

      # 도커
      docker = ["~>1.0"]

      # Include a custom plugin to the bundle. Will search for the plugin in the
      # plugins directory, and package it with the bundle archive. Plugin must have
      # a name of the form: terraform-provider-*, and must be build with the operating
      # system and architecture that terraform enterprise is running, e.g. linux and amd64
      customplugin = ["0.1"]
    }
    ```

이후 os 및 cpu구성을 고려하여 해당 환경설정에 맞는 번들을 만든다.

```sh
terraform-bundle package -os=linux -arch=amd64 terraform-bundle.hcl
```

이렇게 되면 ```terraform_version-bundleYYYYMMDDHH_os_arch.zip``` 형식의 패키지 zip 파일이 나오게 된다.

- ex) terraform_0.12.21-bundle2020030601_linux_amd64.zip

해당 zip 파일을 unzip하면 terrform과 각종 plugin들이 같이 튀어나오는데 해당 프로그램을 통해 init하게되면 packaging 해 둔 provider들은 init 시에 온라인이 아니어도 된다.

---

**P.S**

**terraform 공부 사이트**

- <https://learn.hashicorp.com/terraform#operations-and-development>
