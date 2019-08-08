---

layout: post
comments: true
title: Docker Registry 구축하기 
category: Docker
shortinfo: Docker Registry 구축하고 외부 서버에서 이미지 사용하는 방법 알아보기
tags:
- docker
---

# Docker Registry 구축하기 



## 설치 & 실행

먼저 docker에서 registry 이미지를 받는다.

```shell
docker pull registry 
```

--name 옵션으로 이름을 정하고 --restart를 always로 설정한다. 백그라운드에서 실행되도록 -d 옵션과 -p 옵션으로 포트를 설정하여 이미지를 실행한다.

```shell
docker run --name registry --restart=always -d -p 5000:5000 registry
```



## 이미지

직접 만든 이미지를 사용해도 되고 도커 허브에서 받은 이미지를 사용해도 된다. 여기서는 hello-world를 받도록 한다.

```shell
docker pull hello-world
```

localhost에 docker registry를 5000 포트에서 실행하였으므로 localhost:5000를 이용하여 이미지 이름을 생성한다. docker images로 이미지가 만들어졌는지 확인한 후 푸시하도록 한다.

```shell
docker tag hello-world localhost:5000/hello-world
docker images
docker push localhost:5000/hello-world
```

이미지를 삭제해도 다시 받을 수 있으며 나의 레지스트리는 여기(http://localhost:5000/v2/_catalog)에서 확인할 수 있으며 이미지를 실행할 수 있다.

```shell
docker rmi localhost:5000/hello-world
docker pull localhost:5000/hello-world
docker run localhost:5000/hello-world
```



## openssl 인증서 발급

먼저 ssl 인증서가 필요하다. openssl을 이용해서 인증서를 발급하도록 한다. 아래에서 개인키(key)와 인증요청서(csr)의 이름은 CA로 설정하며 원하는 이름으로 설정하도록 한다. 

- 폴더 생성

```shell
mkdir cert
```

- 개인키 생성

```powershell
openssl genrsa -aes256 -out CA.key 2048
```

- 개인키 권한 설정: group과 other의 permission 제거

```powershell
chmod 600 CA.key
```

- 인증요청서 생성

```powershell
openssl req -new -key CA.key -out CA.csr 
```

- 인증서 생성: 유효기간(365일 설정)

```powershell
openssl x509 -req -days 365 -in CA.csr -signkey CA.key -out CA.crt
```

## Insecure-registries 추가하기

Insecure-registries에 이미지를 추가했던 로컬 서버의 ip를 추가해야 한다. 윈도우에서는 ipconfig 맥과 리눅스에서는 ifconfig로 ip 주소를 확인한다.

- 윈도우 & 맥

윈도우와 맥에서는 도커를 설치할 때 Docker desktop이 설치된다. docker desktop에서 prefrences로 들어간다. 거기서 Daemon에 들어가 확인한 서버의 ip 주소를 5000 포트와 함께 추가한다. (예: 172.16.254.1:5000)

- 리눅스

`/etc/docker` 경로에 들어가면 daemon.json 파일이 있다. `vi daemon.json` 명령어로 아래의 내용을 ip 주소를 수정하여 추가한다. 

```json
{
  "insecure-registries" : [
    "172.16.254.1:5000"
  ]
}
```



## 외부 서버에서 docker registry 사용하기

여기서도 registry 이미지를 받도록 한다. 그 후 registry를 실행할 때 로컬에서 할 때보다 다양한 옵션이 추가된다. -v 옵션에는 인증서가 있는 폴더의 주소를 적어준다. 현재 위치(pwd)에 위에서 만든 cert 폴더가 있다는 가정 하에 아래 명령어를 사용한다. -e에는 registry 환경을 설정한다. cert 폴더에서 개인키와 인증서를 발급받았으므로 각각 cert/CA.key와 cert/CA.crt로 설정한다.

```shell
docker run -d -p 5000:5000 -v $(pwd)/cert \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/cert/CA.crt \ 
-e REGISTRY_HTTP_TLS_KEY=/cert/CA.key \
-e REGISTRY_HTTP_ADDR=localhost:5000 \
--restart=always --name registry registry
```

외부 서버에서 registry를 실행하고 나서 로컬 서버의 registry에 푸시한 이미지를 받도록 하며 ip는 아래와 같다고 가정한다.

```shell
docker pull 172.16.254.1:5000/hello-world
```

