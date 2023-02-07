---
title: "how to write dockerfile"
date: 2022-12-22 00:00:00 -0000
categories: docker gpu
---

# Dockerfile.gpu
- [docker hub](https://hub.docker.com) 에 올라와 있는 official한 기본 저장소를 베이스로 하는 것이 좋다


```bash
## docker hub에서 nvidia의 기본 저장소로 부터 시작한다.
## 엔비디아 그래픽 드라이버, cuda cudnn 등을 기본으로 설치된 상태에서 출발한다.
FROM nvidia/cuda:11.2.1-cudnn8-runtime-ubuntu20.04

## 환경변수를 설정한다
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

## /src 폴더 생성 및 시작 디렉토리 지정
WORKDIR /src

## 로컬에서 있는 파일을 /src 폴더 복사
## docker container 가 동작 하고 있을 때도 복사가 가능하지만 컨테이너 만들떄 넣을 수도 있다
COPY requirements.txt /src/

## 해당 명령어를 실행 RUN
## 업데이트 라이블러리 실행 
## 마지막은 temp 파일을 지워주어 docker를 가볍게 만들어 주기 위해 실행함. (apt auto remove 와 rm)
## apt install -y $(cat linux_packages.txt) 를 이용하길 추천한다
RUN apt-get update && apt upgrade -y && \
      apt install -y  python3-pip libcusolver10  && \
      apt autoremove -y              && \
      rm -rf /var/lib/apt/lists/*

## 필요한 파일들 설치
RUN pip3 install --upgrade pip
RUN pip3 install -r requirements.txt     # install runtime packages
```


# Dockerfile

```bash
## 동작만을 위한 어플리케이션 환경인 컨테이너에서 아래 에디터를 설치하는 것은 지양해야 하지만 필요할땐 사용하자.
$ cat linux_packages.txt
tmux
git
vim
htop

## 버전을 늘 명시 해 줘야 호환성에 의해 설치가 실패하는 일이 줄어든다.
$ cat requirements.txt
numpy==1.2.3
matplotlib==1.2.3

$ cat Dockerfile
FROM tensorflow/tensorflow:2.7.3-gpu

RUN apt-get update && \
      apt install -y $(cat linux_packages.txt)  && \
      apt autoremove -y              && \
      rm -rf /var/lib/apt/lists/*

RUN pip3 install --upgrade pip
RUN pip3 install -r requirements.txt     # install runtime packages
```


## tensorflow official
- [tensorflow official](https://hub.docker.com/r/tensorflow/tensorflow/)
- Tags를 에서 적당한 버전을 선택한다
- 기본적으로 nvidia/cuda에 대한 이미지를 베이스로 하고 있어서 cuda관련 설정이 설치되어 있는 상태이다 
- Jupiter 사용을 하면 기본적으로 도커 컨데이너가 실행 될때 주피터가 실행된다 

```bash
## tensorflow/tensorflow:버전-gpu사용
docker pull tensorflow/tensorflow:2.7.3-gpu
## tensorflow/tensorflow:버전-gpu사용-주피터사용
docker pull tensorflow/tensorflow:2.7.3-gpu-jupyter
```

  - 기본 도커 파일 정보는 [tensorflow GitHub](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/tools/dockerfiles/dockerfiles/devel-gpu.Dockerfile)에 저장소가 잘 변동되기 떄문에 링크가 꺠질수 있다. 해당 부분은 저장소를 잘 찾아 가면 나온다

```bash
## 해당 엔비디아 쿠다를 베이스로 하고 있다.
FROM nvidia/cuda${ARCH:+-$ARCH}:${CUDA}.1-base-ubuntu${UBUNTU_VERSION} as base
```

- 텐서플로우가 필요하면 아래와 유사하게 베이스를 가져 가면 된다.

```bash
FROM tensorflow/tensorflow:2.7.3-gpu
```


## torch official
 - [torch official](https://hub.docker.com/r/pytorch/pytorch)
- Tags를 에서 적당한 버전을 선택한다
- 기본적으로 nvidia/cuda에 대한 이미지를 베이스로 하고 있어서 cuda관련 설정이 설치되어 있는 상태이다 

```bash
## pytorch/pytorch:버전-cuda버전-cudnn버전-사용방법
docker pull pytorch/pytorch:1.9.0-cuda11.1-cudnn8-devel
docker pull pytorch/pytorch:1.8.1-cuda10.2-cudnn7-runtime
```



```bash
FROM pytorch/pytorch:1.9.0-cuda11.1-cudnn8-devel
```