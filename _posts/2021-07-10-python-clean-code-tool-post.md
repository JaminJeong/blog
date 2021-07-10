---
title: "python 코드 정렬 tool 사용하기"
date: 2021-07-10 17:13:00 -0400
categories: python cleancode
---

# python 코드 정리
## IDE 자동 코드 정리 
 * pycharm →ctrl+alt+shift+L :리포멧,자동정렬
 * visual studio code → ctrl+K+F

## import 모듈 정리
### isort
 * 설치 : pip install isort
 * 사용법 : isort test.py
 * <https://pypi.org/project/isort/>

## 사용하지 않는 import 모듈과 변수 정리
### autoflake
 * 설치 : pip install autoflake
 * 사용법 : autoflake --in-place --remove-unused-variables Example.py ← 불필요한 import 모듈 삭제와 불필요한 변수 삭제
 * 사용법 : autoflake --in-place Example.py ← 불필요한 import 모듈만 삭제
 * <https://python.plainenglish.io/autoflake-remove-unused-imports-unused-variables-from-python-code-4774c1117099>
 * git을 이용하여 정렬 및 코드 내용 확인 후 push 

## f-string 변환
### flynt
 * 설치 : pip install flynt
 * 사용법 : autoflake ./
 * 사용법 : autoflake ./test.py
 * <https://github.com/ikamensh/flynt>
 * "{}".format(test) → f"{test}" 변환

## 코드 자동 정렬
### black
 * 설치 : pip install git+git://github.com/psf/black
 * 사용법 : black ./
 * 사용법 : black ./test.py
 * <https://github.com/psf/black>
 * python 코드 정렬 기준 (<https://www.python.org/dev/peps/>) 에 따라 자동 정렬

# Crontab을 이용한 자동정렬 업데이트
### crontab
 * 새벽 4시 마다 동작함.

```bash
0 4 * * * bash /your/git/folder/clean_code.sh /your/git/folder/
```

### 자동 shell code
 * clean_code.sh

```bash
target_dir="$1"
if [ -d "${target_dir}" ]; then
    echo "Auto commit: ${target_dir}"
    cd "${target_dir}"
 
    # git clean
    echo "git clean"
    git clean -d -f
    git status | grep "modified:" | awk '{print "git checkout -- "$2 }' | sh
    git status | grep "deleted:" | awk '{print "git checkout -- "$2 }' | sh
 
    echo "git checkout yourbranch"
    git checkout yourbranch
    echo "git pull"
    git pull
 
    # code clean
    echo "[clean code] autoflake"
    find ./ | grep .py | grep -v .pyc | awk '{print "autoflake --in-place "$1 }' | sh
    echo "[clean code] isort"
    find ./ | grep .py | grep -v .pyc | awk '{print "isort "$1 }' | sh
    echo "[clean code] flynt"
    flynt ./
    echo "[clean code] black"
    black ./
 
    # git upload
    echo "git push"
    git add ./
    git commit -m "[clean code] auto commit"
    git push origin yourbranch
 
    echo "DONE!"
fi

```
