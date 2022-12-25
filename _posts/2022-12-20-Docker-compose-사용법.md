---
title: "docker compose"
date: 2022-12-22 00:00:00 -0000
categories: docker docker-compose
---

# why? docker-compose?
- docker run의 경우 아래와 같이 어떤 shell 파일로 관리를 해야하는 불편함이 있고
- 사용자는 어떤 파일을 실행해야 컨테이너를 실행 할 수 있을지 알수 없다. 
- 특히 다른 여러가지 서비스와 연합하여 동작하는 경우에는 따로 docker run을 순서에 맞게 잘 작성 해 줘야 한다.
- docker-compose 는 이런 서비스간 의존성을 해결해 줄 뿐 아니라 동일 네트워크로 묶어 준다.
- 자세한 정보는 [생활코딩](https://www.youtube.com/c/생활코딩1)의 [Docker compose 를 이용해서 복잡한 도커 컨테이너를 제어하기](https://www.youtube.com/watch?v=EK6iYRCIjYs)를 참고하자

# docker-compose? 왜 씀? <- 블로그 발췌
1. docker 보다 간결함
 
간단한 html을 만들고 nginx로 연결하려면 아래와 같이 docker 명령어를 입력해야 한다.
컨테이너 80 포트를 로컬 8080 포트와 매핑하고, 종료시 컨테이너를 삭제하기 위해 --rm 옵션을 주었고
로컬의 특정 경로의 폴더 혹은 파일을 참고하도록 volume 옵션을 통해 현재 경로 $(pwd)를 nginx 컨테이너 내에 /usr/share/nginx/html에 매핑해주자.
이 모든 과정은 이해할 수는 있지만 살짝 verbose하다. compose를 사용하면 이런 번거로운 cli를 작성하지 않고, 
docker-compose.yml을 활용할 수 있다.

```bash
$ docker run -it -p 8080:80 --rm -v $(pwd):/usr/share/nginx/html/ nginx
```
2. 컨테이너 간 연결이 쉬워진다.
 
아래는 postgres와 django-sample이란 컨테이너를 연결한 것이다.
--link 옵션을 주어서 django-sample 컨테이너에게 db라는 이름으로 postgres 컨테이너의 존재를 알린 것이다.
{연결할 컨테이너 이름}:{해당 컨테이너에서 참고할 이름}
- 역시나 verbose하다.
```bash
docker run --rm -d --name postgres \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres

docker run -d --rm \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```
3. 특정 컨테이너끼리만 통신할 수 있는 가상 네트워크 환경을 관리하는데 너무 명령어가 길어진다.
 
--network 옵션을 통해서 특정 네트워트 내에만 존재하는 컨테이너끼리만 통신할 수 있도록 만들어주었다.
django2의 경우 해당 네트워크 내에 존재하지 않는 컨테이너이기 때문에 --link로 연결해도 통신할 수 없게 된다.
 
너무 verbose하다...
```bash
// network 생성
docker network create --driver bridge web-service

// 해당 network를 활용하여 컨테이너 실행
docker run --rm -d --name postgres \
  --network web-service \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres

docker run -d --rm --name django1 \
  --network web-service \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample

docker run -d --rm --name django2 \
  -p 8001:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```

## docker-compose를 구성하기 위한 yml 파일을 만들자. 
- 여기서 depends_on 로 다른 서비스에 대한 의존성을 만들 수 있음에 주목하자.
```bash
version: '3'

volumes:
  postgres_data: {}

services:
  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgres/data
    environment:
      - POSTGRES_DB=djangosample
      - POSTGRES_USER=sampleuser
      - POSTGRES_PASSWORD=samplesecret
      
  django:
    build:
      context: .
      dockerfile: ./compose/django/Dockerfile-dev
    volumes:
      - ./:/app/
    command: ["./manage.py", "runserver", "0:8000"]
    environment:
     - DJANGO_DB_HOST=db
    depends_on:
      - db
    restart: always
    ports:
      - 8000:8000
```

## tensorflow를 이용한 docker run vs docker-compose
### before
```bash
## docker run [OPTION] IMAGE[:TAG] [COMMAND]
$ docker run -it \
        --rm \                                   # 컨테이너가 동작을 멈추면 삭제 되는 옵션
        --gpus all \                             # GPU 설정
        --gpus "device=0" \                      # GPU 숫자로 설정
        --name container_name \                  # container 이름 설정
        -w /root/project \                       # container 시작 working directory 설정
        -v /home/$USER/project:/root/project \   # container 외/내부 폴더 Volume mount 하기
        -p 11111:11111 \                         # 이미 사용하고 있는 port는 피하자 (docker ps -a)
        -p 22222:22222 \                         # port 수는 65536 이하로.
        -p 22-25:22-25 \                         # 여러개의 포트를 사용할 때 사용
        tensorflow/tensorflow:2.8.2-gpu \        # 이미지 이름
        /bin/bash                                # 시작하면서 동작할 명령을 설정할 수 있음.
```

### after
- docker-compose.yml 파일을 아래처럼 작성하자
- stdin_open: true와 tty: true는 도커 컨포즈의 철학과는 다를 수 있지만 필요하다면 사용하자.
```yaml
version: "3.7"                              # version 정보

services:
  tensorflow_2:
    image: tensorflow/tensorflow:2.2.0-gpu  # 이미지 이름
    container_name: jmjeong_test            # --name container_name
    stdin_open: true                        # docker run -i 
    tty: true                              # docker run -t
    command:
      - /bin/bash                           # 시작하면서 동작할 명령어
    entrypoint: /bin/bash                   # command와 동일 시작하면서 동작할 명령어
    working_dir: /root/test                 # -w 옵션 동일
    ports:
      - 9212:9211                            # 포트설정
    volumes:
      - /home/$USER/test:/root/test
    deploy:                                  # GPU 설정
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["1"]
              capabilities: [gpu, utility]
```

## container 동작
- 이 명령어를 이용하여 주고 도커를 실행하고 끈다고 생각하자
```bash
## docker run처럼 동작
$ docker-compose up 
$ docker-compose up -d # detach 로 동작
## docker stop + docker rm container_name 으로 동작
$ docker-compose down
```

## container 상태보기
```bash
$ docker-compose ps
$ docker-compose ps -a
    Name        Command    State                    Ports                  
---------------------------------------------------------------------------
jmjeong_test   /bin/bash   Up      0.0.0.0:9212->9211/tcp,:::9212->9211/tcp
```

## docker-compose 명령어
```bash
docker-compose up -d // 도커 백그라운드 실행
docker-compose up --force-recreate // 도커 컨테이너 새로 만들기
docker-compose up --build // 도커 이미지 빌드 후 compose up
```

```bash
docker-compose start // 정지한 컨테이너를 재개
docker-compose start mysql // mysql 컨테이너만 재개

docker-compose restart // 이미 실행 중인 컨테이너 다시 시작
docker-compose restart redis // 이미 실행중인 redis 재시작

docker-compose stop // gracefully stop함.
docker-compose stop wordpress

docker-compose down // stop 뿐만 아니라 컨테이너 삭제까지

docker-compose logs
docker-compose logs -f // 로그 watching

docker-compose ps // 컨테이너 목록

docker-compose exec [컨테이너] [명령어]
docker-compose exec wordpress bash // wordpress에서 bash 명령어 실행

docker-compose build // build 부분에 정의된 대로 빌드
docker-compose build wordpress // wordpess 컨테이너만 빌드

docker-compose run [service] [command] // 이미 docker-compose 가동 중인 것과 별개로 하나 더 올릴 때
docker-compose run nginx bash
```

## docker-compose build
- 이미지를 자체 빌드 후 사용할 경우 build를 이용할 경우에 사용합니다. 이미지 빌드를 위한 dockerfile이 필요하니까 지정해주면 됩니다.
- 자체 빌드니까 image 속성 대신 사용합니다.
- docker-compose build를 통해서 빌드한 후 docker-compose up해주면 된다.
```bash
services:
  django:
    build:
      context: .
      dockerfile: ./compose/django/Dockerfile-dev 
  postgres:
  	image: ....
```

# 우리가 할 수 있는 구성
- 학습 환경, tensorboard, mlflow, 등 다양한 환경과 어울어져 사용할 수 있음.
- [딥러닝을 위한 docker-compose 설치 및 사용법](https://hanseokhyeon.tistory.com/entry/딥-러닝을-위한-docker-compose-설치-및-사용법-pytorch-tensorboard-예제)을 참고하자
- train을 tensorboard에 대한 의존성이 있게 만들 수 있다.

# reference
- [docker docs gettingstarted](https://docs.docker.com/compose/gettingstarted/)
- [도커 컴포즈를 활용하여 완벽한 개발 환경 구성하기](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#그런데-앱-서비스에서-db-서비스를-어떻게-찾았지)
- [docker compose와 간단한 compose 문법](https://darrengwon.tistory.com/793)
- [딥러닝을 위한 docker-compose 설치 및 사용법](https://hanseokhyeon.tistory.com/entry/딥-러닝을-위한-docker-compose-설치-및-사용법-pytorch-tensorboard-예제)
