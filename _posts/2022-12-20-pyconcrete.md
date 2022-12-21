---
title: "Pyconcrete"
date: 2022-12-22 00:00:00 -0000
categories: Pyconcrete
---

# Pyconcrete
- pyconcrete는 python 소스를 암호화 해주는 툴이다
- 파이썬을 실행하거나 빌드 할때 생성되는 .pyc 파일을 .pye 파일로 변환하여 암호화된 실행 환경을 제공한다
- 원본 소스 코드를 공개하지 않고 암호화된 실행 파일들만 제공할 수 있다
- pytroch, tensorflow 환경에서 동작 여부는 아직 확인되지 않음.
- Github : [pyconcrete](https://github.com/Falldog/pyconcrete)

# Dockerfile
```bash
# torch 기본 이미지에서 시작
FROM pytorch/pytorch:1.7.1-cuda11.0-cudnn8-devel

ARG DEBIAN_FRONTEND=noninteractive

# working 폴더 설정
WORKDIR /workspace

# 업데이트
RUN ( apt update | true ) && apt install -y python3.8-dev && rm -rf /var/lib/apt/lists/*

# requirements.txt가 있는 경우 pip install 로 설치
COPY requirements.txt .
RUN pip3 install -U pip && pip3 install -U setuptools wheel && pip3 install -r requirements.txt

# 소스코드 복사
COPY . .
ARG PYCONCRETE_PASSPHRASE
# pyconcrete 설치 PYCONCRETE_PASSPHRASE는 암호키
RUN PYCONCRETE_PASSPHRASE=$PYCONCRETE_PASSPHRASE pip install pyconcrete

# 학습 파일 암호화 
RUN pyconcrete-admin.py compile --source=/workspace/train.py --pye && rm -v /workspace/train.py
RUN pyconcrete-admin.py compile --source=/workspace/model.py --pye && rm -v /workspace/model.py
# 기존 python 파일 삭제
RUN find . | grep -E "(/__pycache__$|\.pyc$|\.pyo$)" | xargs rm -rf
# .git을 삭제하여 소스코드 수정 기록 삭제
# docker file 삭제
RUN rm -rf /workspace/.git* Dockerfile
```

# Install
```bash 
$ PYCONCRETE_PASSPHRASE=<your passphrase here> pip install pyconcrete
$ pip install pyconcrete --egg --install-option="--passphrase=<your passphrase>"
```

# Usage
- .py 파일들을 .pye 파일로 변환한다
```bash 
$ pyconcrete-admin.py compile --source=<your py script>  --pye
$ pyconcrete-admin.py compile --source=<your py module dir> --pye
```

### 일부분 암호화
  - pyconcrete 다운로드 후 setup.py 설치
```bash
$ python setup.py install \
  --install-lib=<your project path> \
  --install-scripts=<where you want to execute pyconcrete-admin.py and pyconcrete(exe)>
```
  - main 스크립트에 pyconcrete를 import함
```bash
main.py       # import pyconcrete and your lib
pyconcrete/*  # put pyconcrete lib in project root, keep it as original files
src/*.pye     # your libs
```

- 실행
```bash
$ pyconcrete main.pye
src/*.pye  # your libs
```

# Release
- 배포시 기존의 .py파일을 삭제하고 git으로 관리 되고 있다면 .git 파일을 삭제하여 소스 파일에 대한 정보를 완전히 삭제 해 준다
