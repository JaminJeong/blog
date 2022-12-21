---
title: "pyscaffold"
date: 2022-12-22 00:00:00 -0000
categories: pyscaffold
---

# installation
## basic installation 
```bash
$ pip install --upgrade pyscaffold
``` 
## optional installation
```bash
## package화를 위한 기본 설치
$ pip install tox setuptools setuptools_scm wheel
## rst파일이 아닌 markdown 을 사용하기 위한 페키지 설치
$ pip install pyscaffoldext-markdown
``` 

## create projects
  - 기본적으로 우리 모두는 rst 파일에 익숙하지 않기 때문에 --markdown 옵션을 넣어서 생성해야 한다.
```bash 
$ putup my_project

## interactive 하게 설정을 조정하면서 생성할 수 있다.
$ putup -i my_project

## 설명에 따른 문서를 모두 markdown으로 생성한다.
## pyscaffoldext-markdown 패키지가 설치 되어 있어야 한다
$ putup my_project --markdown
```
  - 나머지 생성법은 [pyscaffold](https://pyscaffold.org/en/stable/usage.html) 여기를 참조하자.

- 생성된 my_project 폴더 구조는 아래와 같다
- docs 폴더와 src 폴더가 생성되었다. 우리가 만드는 소스는 src에 저장한다.
- setup.py와 setup.cfg 파일을 수정하여 패키지와 하자 (해당 부분은 이후 추가 예정)
```bash
$ cd my_project
$ tree
.
├── AUTHORS.md
├── CHANGELOG.md
├── CONTRIBUTING.rst
├── LICENSE.txt
├── README.md
├── docs
│   ├── Makefile
│   ├── _static
│   ├── authors.md -> ../AUTHORS.md
│   ├── changelog.md -> ../CHANGELOG.md
│   ├── conf.py
│   ├── contributing.rst
│   ├── index.md
│   ├── license.rst
│   ├── readme.md -> ../README.md
│   └── requirements.txt
├── pyproject.toml
├── setup.cfg
├── setup.py
├── src
│   └── test1
│       ├── __init__.py
│       └── skeleton.py
├── tests
│   ├── conftest.py
│   └── test_skeleton.py
└── tox.ini

5 directories, 22 files
```

## docs
- 충분히 관련 문서를 정리 한 다음 배포하도록 하자
- docs 폴더를 빌드하여 html 파일들을 생성해 준다
```bash
$ tox -e docs  # to build your documentation
```
```bash
## 문서 파일 생성 전
$ tree docs
docs
├── Makefile
├── _static
├── authors.md -> ../AUTHORS.md
├── changelog.md -> ../CHANGELOG.md
├── conf.py
├── contributing.rst
├── index.md
├── license.rst
├── readme.md -> ../README.md
└── requirements.txt

## 생성
$ tox -e docs  
.......
  docs: commands succeeded
  congratulations :)
## 생성 후
$ tree docs
docs
├── Makefile
├── _build
│   ├── doctrees
│   │   ├── api
│   │   │   ├── modules.doctree
│   │   │   └── test1.doctree
│   │   ├── authors.doctree
│   │   ├── changelog.doctree
│   │   ├── contributing.doctree
│   │   ├── environment.pickle
│   │   ├── index.doctree
│   │   ├── license.doctree
│   │   └── readme.doctree
│   └── html
│       ├── _modules
│       │   ├── index.html
│       │   └── test1
│       │       └── skeleton.html
│       ├── _sources
│       │   ├── api
│       │   │   ├── modules.rst.txt
│       │   │   └── test1.rst.txt
│       │   ├── authors.md.txt
│       │   ├── changelog.md.txt
│       │   ├── contributing.rst.txt
│       │   ├── index.md.txt
│       │   ├── license.rst.txt
│       │   └── readme.md.txt
│       ├── _static
│       │   ├── alabaster.css
│       │   ├── basic.css
│       │   ├── custom.css
│       │   ├── doctools.js
│       │   ├── documentation_options.js
│       │   ├── file.png
│       │   ├── jquery-3.5.1.js
│       │   ├── jquery.js
│       │   ├── language_data.js
│       │   ├── minus.png
│       │   ├── plus.png
│       │   ├── pygments.css
│       │   ├── searchtools.js
│       │   ├── underscore-1.13.1.js
│       │   └── underscore.js
│       ├── api
│       │   ├── modules.html
│       │   └── test1.html
│       ├── authors.html
│       ├── changelog.html
│       ├── contributing.html
│       ├── genindex.html
│       ├── index.html
│       ├── license.html
│       ├── objects.inv
│       ├── py-modindex.html
│       ├── readme.html
│       ├── search.html
│       └── searchindex.js
├── _static
├── api
│   ├── modules.rst
│   └── test1.rst
├── authors.md -> ../AUTHORS.md
├── changelog.md -> ../CHANGELOG.md
├── conf.py
├── contributing.rst
├── index.md
├── license.rst
├── readme.md -> ../README.md
└── requirements.txt
```

## build
- 패키지를 build 하여 dist 폴더에 whl 설치 파일을 생성한다. 
```bash
$ tox -e build  # to build your package distribution
  build : commands succeeded
  congratulations :)
$ tree dist 
dist
├── test1-0.0.post1.dev1+g68ff46f-py3-none-any.whl
└── test1-0.0.post1.dev1+g68ff46f.tar.gz
```
- 아래와 동작이 동일하다 
```bash
$ python setup.py bdist_wheel
```

## publish
- 설정 파일에 GitHub 주소(pypi)로 만든 패키지와 문서를 배포 한다.
- 해당 pypi 저장소는 setup.cfg에서 url을 수정해 줘야 함. <= 차후 업데이트
```bash
$ tox -e publish  # to test your project uploads correctly in test.pypi.org
$ tox -e publish -- --repository pypi  # to release your package to PyPI

## help 옵션
tox -av  # to list all the tasks available
```

# install package
```bash
$ pip install -e .
```

# test module
- to be updated....

# file structure
- 우리가 작업할 코드들은 src 폴더 아래에 모두 넣어서 사용함.
- 각종 파일 구조와 사용법에 대해서는 확인 중임.
- to be updated....

# setup.py 설정하는 방법
- to be updated....

# reference 
  - [pyscaffold](https://pyscaffold.org/en/stable/usage.html)
  - [pyscaffoldext-markdown](https://github.com/pyscaffold/pyscaffoldext-markdown)
  - [Python project management and construction tools](https://pythonmana.com/2021/10/20211031005016506u.html)
  - [tox](https://tox.wiki/en/latest/)