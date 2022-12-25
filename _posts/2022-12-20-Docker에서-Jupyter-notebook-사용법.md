---
title: "docker"
date: 2022-12-22 00:00:00 -0000
categories: docker jupyter
---

# Docker Jupyter-notebook 사용법

- Docker를 이용, Jupyter editor를 사용하는 방법입니다.
- Node2를 기준한 예제이며, Node 및 NIPA도 같은 방향으로 사용이 가능합니다.

> <Docker 및 Linux 관련 내용의 코드>
>
> 1. [Docker 란](https://git.testdns.dev/documents/home/-/wikis/Docker)
> 2. [Docker 기본](https://git.testdns.dev/documents/home/-/wikis/Docker-%EA%B8%B0%EB%B3%B8-%EC%82%AC%EC%9A%A9%EB%B2%95)
> 3. [Docker 파일](https://git.testdns.dev/documents/home/-/wikis/Dockerfile-%EC%9E%91%EC%84%B1%EB%B2%95)

- 간단한 예제로, 순서대로 적어두었습니다.
- 예제 진행은 mac-os로 진행하였지만, 진행 단계는 동일합니다.

---

## 1. Node/NIPA의 Local에 Docker 폴더 설정

#### 1. Terminal (CMD) 실행

Terminal (CMD) 를 실행해서 Node/NIPA에 접속합니다.

`ssh username@{node/nipa@}.gpu.testdns.dev`

#### 2. 접속된 후, Docker 파일이 설치될 폴더를 생성합니다

`mkdir project`

#### 3. 생성된 project 폴더로 directory를 이동합니다.

`cd project`

* 현재 folder tree

```
.
├── project       <<<<<<
```

--------------------------------



## 2. Docker 설치 파일 (docker-run.sh) 작성

#### 1. project 폴더내에서 sh 파일 생성

`vi docker_run.sh`

#### 2. docker_run.sh 파일 내 내용 작성

* `i` 를 누르면 맨 밑 하단에 `--insert--` 로 바뀌어야 sh 작성이 가능합니다.


* container_name에는 가급적 계정 아이디를 포함하여 누구의 job인지 알수 있게 하자.


* **작성예제**
> <details><summary>    [모든 GPU 사용시]    </summary>
> 
> ```
> docker run -it \
>         --gpus all \                             # GPU 설정
>         --name {container_name} \                     # container 이름 설정
>         -v /home/{user_name}/project:/root/project \      # container 외/내부 위치 설정
>         -v /mnt/dataset/{USING_DATAFOLDER}:/root/project/{LINK_FOLDER} \ # NAS에서 사용할 데이터 연동
>         -p 00000:00000                           # port 설정. > 3개 정도가 적당하다. 
>         -p 11111:11111                           # 이미 사용하고 있는 port는 피하자 (docker ps -a)
>         -p 22222:22222                           # port 수는 65536 이하로.
>         tensorflow/tensorflow:2.8.2-gpu-jupyter  # Docker 로컬 이미지에 존재하면
>                                                  # 그 이미지를 사용하고 존재하지 않으면 
>                                                  # docker hub에서 pull 하여 사용함.
>                                                  # 혹은 자신이 빌드한 이미지를 사용함.
>                                                    ~ -jupyter는 굳이 필요는 없음 (재설치 경우 존재)
> 
> ```
> 
> </details>
> 
> <details><summary>    [특정 GPU 사용시]    </summary>
> 
> ```
> docker run -it \
>         --gpus device=1 \                             # 1번 GPU 설정 (0,1 ... 등 여러개 가능)
>         --name {container_name} \                     # container 이름 설정
>         -v /home/{user_name}/project:/root/project \      # container 외/내부 위치 설정
>         -v /mnt/dataset/{USING_DATAFOLDER}:/root/project/{LINK_FOLDER} \ # NAS에서 사용할 데이터 연동
>         -p 00000:00000                           # port 설정. > 3개 정도가 적당하다. 
>         -p 11111:11111                           # 이미 사용하고 있는 port는 피하자 (docker ps -a)
>         -p 22222:22222                           # port 수는 65536 이하로.
>         tensorflow/tensorflow:2.8.2-gpu-jupyter  # Docker 로컬 이미지에 존재하면
>                                                  # 그 이미지를 사용하고 존재하지 않으면 
>                                                  # docker hub에서 pull 하여 사용함.
>                                                  # 혹은 자신이 빌드한 이미지를 사용함.
>                                                    ~ -jupyter는 굳이 필요는 없음 (재설치 경우 존재)
> 
> ```
> 
> </details>
> 


* 작성이 완료되었으면 `esc` 후 `:wq` 

* 현재 folder tree

```
.
├── project
│   ├── docker_run.sh                <<<<<<
```

--------------------------------


## 3. docker 설치

#### 1. docker_run.sh 파일을 실행하여 docker 설치
`bash docker_run.sh`

* 시간이 좀 걸린다. :disappointed: 

* 현재 folder tree

```
.
├── project
│   ├── data
│   ├── docker_run.sh
└── snap                              <<<<<<
    └── node
        ├── @@@@
        ├── common
        └── current -> @@@@
```

--------------------------------

## 4. docker 접속

#### 1. `bash docker_run.sh` 로 설치된 docker에 접속한다.

`docker exec -it {container_name} /bin/bash`

* Docker exit 상태시 > `docker start {container_name}`

--------------------------------

## 5. Jupyter Notebook 실행 파일 작성

#### 1. project 폴더내에서 sh 파일 생성 

`vi jupyter.sh`
 
#### 2. jupyter.sh 파일 내 내용 작성

* `i` 를 누르면 맨 밑 하단에 `--insert--` 로 바뀌어야 sh 작성이 가능합니다.

```
#export CUDA_VISIBLE_DEVICES=1
jupyter-notebook -—notebook-dir=/root/project -—ip 0.0.0.0 -—allow-root —-port @@@@@@ # 설정한 포트
```

* 작성이 완료되었으면 `esc` 후 `:wq` 

* 현재 folder tree

```
.
├── project
│   ├── data
│   ├── docker_run.sh
│   ├── jupyter.sh                  <<<<<<
└── snap
    └── node
        ├── @@@@
        ├── common
        └── current -> @@@@

```

--------------------------------

## 6. Jupyter notebook token, port 연결

#### 1. `jupyter.sh` 파일을 실행하여 token 얻기

`bash jupyter.sh`

#### 2. Token 복사 및 Node/NIPA의 Port 찾기

   1. `http:~~~~~~~~~~~token:##############`의  token을 복사
   2. Node/NIPA Port > @@.@.@@.@@


--------------------------------

## 7. Jupyter notebook 실행

* Jupyter 가 Terminal (CMD)에서 실행된 상태

#### 1. Browser URL 에서 실행

> URL > `{NODE/NIPA Port}:{USER_PORT}`
>
> EX  > `@@.@.@@.@@:00000`

#### 2. Jupyter notebook 실행. Token 및 password 설정

> Token > ###################
>
> PWD   > @@@@@@@@@@@@@@@@@@@

--------------------------------

## 3. 할당 GPU 확인

* Jupyter Notebook이 실행된 상태

#### 1. Terminal 혹은 Jupyter Notebook ipynb 에서 실행

```
from tensorflow.python.client import device_lib
device_lib.list_local_devices()
```

#### 2. Docker 내에서 Terminal (CMD) 실행

`nvidia-smi`

* 동일한지 확인한다.
* GPU를 하나만 선택할 경우, `GPU index = 0` 으로 나온다.

* 확인
>
> `Node/NIPA내 nvidia-smi`
>
> ![스크린샷_2022-05-26_17.15.37](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/04b0d9bd0432a4059c716a1960eabaa2/스크린샷_2022-05-26_17.15.37.png)
>
> `Jupyter Notebook > GPU 1 설정`
>
> ![스크린샷_2022-05-26_17.08.57](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/81e7296ebf5df4d28e23495da72389e4/스크린샷_2022-05-26_17.08.57.png)
>
> `Terminal (CMD) `
>
>![스크린샷_2022-05-26_17.11.13](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/20f5bd008b3c82fa3007d34d4ea03343/스크린샷_2022-05-26_17.11.13.png)
>

## 4. Jupyter Notebook 보단 Jupyter Lab을 원한다면

#### 1. Jupyter Lab 설치

* Docker Container에 접속한 상태에서

`pip install jupyterlab`

* `pip list | grep jupyter` 로 `jupyterlab`이 설치되었는지 확인한다.

>![스크린샷_2022-06-02_09.27.37](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/0c946a532a2ee207f04d6456f52fdf07/스크린샷_2022-06-02_09.27.37.png)

#### 2. Jupyter Lab 실행

* Docker Container에 접속한 상태에서 실행
* Notebook이 Background에서 실행되어 있으면 안됨. > Notebook 켜짐..

`jupyter lab --allow-root --ip 0.0.0.0 --port @@@@@ # jupyter notebook에서 쓰인 port`

* 이후 jupyter notebook 사용과 동일하다.
* **jupyter notebook 실행파일처럼 script파일을 만들어서 사용하면 편리하다**

>
>`vi lab.sh`
>
>```
>jupyter lab --allow-root --ip 0.0.0.0 --port @@@@@ # jupyter notebook에서 쓰인 port
>```
>
>`esc`
>
>`:wq`
>
>* docker container 내 에서
>
>`bash lab.sh`