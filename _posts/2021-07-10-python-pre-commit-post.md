---
title: "python pre-commit"
date: 2021-07-10 19:38:10 -0400
categories: python git pre-commit
---

## 목적
Git의 pre-commit 은 commit 하기 전에 원하는 동작을 실행해 주는 훌륭한 도구입니다 
Python의 경우 필요하다고 판단되는 도구들을 정리해 봤습니다

## 출처
 * 블로그
   - http://snowdeer.github.io/git/2021/04/27/use-git-pre-commit/
   - https://www.daleseo.com/pre-commit/
 * 제공되는 hook
   - https://pre-commit.com/hooks.html

## 설치
 * pip install pre-commit
 * pre-commit-config.yaml 파일 생성

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.0.1
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
 
# flake의 경우 문법을 너무 자세히 검사하여 커밋이 되지 않은 필요 하면 사용
# 파일의 어색한 부분을 알려주고 자동 변환 해주지 않음
#-   repo: https://github.com/PyCQA/flake8
#    rev: 3.9.2
#    hooks:
#    -   id: flake8
#        additional_dependencies: [flake8-bugbear]
 
# black 은 python의 가독성을 향상 시키는 정리하는 모듈
# 파일을 자동 변환 함.
-   repo: https://github.com/psf/black
    rev: '21.6b0'
    hooks:
    -   id: black
 
 
# flynt 은 .format()을 f"" 형태로 변환
# 개행이 되어 있는 문법은 수정을 못해주니 확인해야 함.
# 파일을 자동 변환 함.
-   repo: https://github.com/ikamensh/flynt
    rev: '0.63'
    hooks:
    -   id: flynt
 
# isort는 import 모듈을 정리해줌
# 파일을 자동 변환 함.
-   repo: https://github.com/PyCQA/isort
    rev: '5.9.2'
    hooks:
    -   id: isort
 
# autoflake는 사용하지 않는 import 모듈을 제거 함.
# 파일을 자동 변환 함.
-   repo: https://github.com/myint/autoflake
    rev: 'v1.4'
    hooks:
    -   id: autoflake
        args: [--in-place]
```

## pre-commit-config.yaml  파일 구조

```yaml
-   github 저장소 위치
    rev: revision (git tag를 통해 revision을 복붙함.)
    hooks:
    -   id: autoflake (이름)
        args: [--in-place] (입력인자 필요한 경우 입력)
 
 
-   repo: https://github.com/myint/autoflake
    rev: 'v1.4'
    hooks:
    -   id: autoflake
        args: [--in-place]
```

## 설정
```bash
## 샘플 파일 생성
$ pre-commit sample-config > .pre-commit-config.yaml
 
## yaml의 git 주소를 통해 자동으로 precommit 검사를 업데이트함.
$ pre-commit autoupdate
 
## 수동으로 실행
$ pre-commit run
 
## run을 실행 시키지 않아도 commit 마다 실행
## 최초 한번 실행
$ pre-commit install
```

## 사용법
```bash
## 테스트 파일 생성
$ cat test.py
import numpy as np
print('hello world')
 
$ git add test.py
$ git commit -m "test"
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...........................................(no files to check)Skipped
Check for added large files..............................................Passed
black....................................................................Failed
- hook id: black - files were modified by this hookreformatted test.py  All done!
1 file reformatted.1
flynt....................................................................Passed
isort....................................................................Passed
autoflake................................................................Passed
 
## black 이 '' 를 ""로 변경한 것을 확인 할 수 있음
$ git diff ./test.py
-print('hello world')
+print("hello world")
 
## commit 완료
$ git add ./test.py
$ git commit -m "test"
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...........................................(no files to check)Skipped
Check for added large files..............................................Passed
black....................................................................Passed
flynt....................................................................Passed
isort....................................................................Passed
autoflake................................................................Passed
```
