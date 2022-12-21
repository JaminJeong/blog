---
title: "python"
date: 2022-12-22 00:00:00 -0000
categories: python ci/cd gitlab
---

# python CI/CD
- 다양한 역할을 수행하게 된다
  - test build
  - package build
  - docs build

# gitlab이 기본 생성하는 파일
- 3가지 과정을 통해 ci/cd를 수행한다는 것을 알수 있다
  - build: 소스르 빌드 합니다. c++이나 java 같이 스크립트 언어가 아닌 경우는 빌드가 꼭 필요하다
    - build-job
  - test: unit 테스트와 lint 테스트. python 과 같이 소스 빌드가 없는 경우 더욱 유용하다
    - unit-test-job: [TDD](https://en.wikipedia.org/wiki/Test-driven_development)를 사용하는 모든 코드는 Test 코드를 만들고 테스트 과정을 거친다. 논리적인 오류를 검증한다
    - lint-test-job: 코딩 컨벤션을 검사하여 clean code를 유지 시킨다. 예를들면 [clean code](clean code tool)의 flake8 과 같은 문법 검사이다
  - deploy
    - deploy-job: python의 경우 package파일(.whl)을 만들고 pypi에 배포하는 과정이다

- Linting은 잠재적 오류에 대한 코드를 분석하는 프로그램을 실행하는 프로세스

```yaml
# This file is a template, and might need editing before it works on your project.
# To contribute improvements to CI/CD templates, please follow the Development guide at:
# https://docs.gitlab.com/ee/development/cicd/templates.html
# This specific template is located at:
# https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Getting-Started.gitlab-ci.yml

# This is a sample GitLab CI/CD configuration file that should run without any modifications.
# It demonstrates a basic 3 stage CI/CD pipeline. Instead of real tests or scripts,
# it uses echo commands to simulate the pipeline execution.
#
# A pipeline is composed of independent jobs that run scripts, grouped into stages.
# Stages run in sequential order, but jobs within stages run in parallel.
#
# For more information, see: https://docs.gitlab.com/ee/ci/yaml/index.html#stages

stages:          # List of stages for jobs, and their order of execution
  - build
  - test
  - deploy

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."

unit-test-job:   # This job runs in the test stage.
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - sleep 60
    - echo "Code coverage is 90%"

lint-test-job:   # This job also runs in the test stage.
  stage: test    # It can run at the same time as unit-test-job (in parallel).
  script:
    - echo "Linting code... This will take about 10 seconds."
    - sleep 10
    - echo "No lint issues found."

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."
```

# gitlab의 python에 대해 제공하는 기본설정
- venv를 이용하여 가상환경에서 필요한 패키지를 설치한다
  - tox, flake8, sphinx 등.. 각 설명은 주석에 달도록함
- 전체과정
  - before script: 실행전에 환경설정을 해준다.
  - test: 위에서 이야기한 unitest와 Lint를 수행함.
  - run: 해당 소스를 패지화 한다. dist 폴더에 whl 파일로 배포하고 설치한다
  - pages: 각종 설명파일들. readme.md, contribute.md, license.md 등을 html형태의 배포 웹형태로 변경한다.
- **결론적으로 우리는 pyscaffold와 docker 환경을 사용하기 때문에 해당 방식은 사용하지 않는다. 하지만 배울점이 많다.**
- 해당 스크립트는 셋업 상태에 따라 실행되지 않는 문제도 있다.
- [gitlab-ci default python file](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Python.gitlab-ci.yml)

```yaml
# To contribute improvements to CI/CD templates, please follow the Development guide at:
# https://docs.gitlab.com/ee/development/cicd/templates.html
# This specific template is located at:
# https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Python.gitlab-ci.yml

# Official language image. Look for the different tagged releases at:
# https://hub.docker.com/r/library/python/tags/
image: python:latest

# Change pip's cache directory to be inside the project directory since we can
# only cache local items.
variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

# Pip's cache doesn't store the python packages
# https://pip.pypa.io/en/stable/topics/caching/
#
# If you want to also cache the installed packages, you have to install
# them in a virtualenv and cache it as well.
cache:
  paths:
    - .cache/pip
    - venv/

before_script: # 환경설정 virtualenv를 사용함.
  - python --version  # For debugging
  - pip install virtualenv
  - virtualenv venv
  - source venv/bin/activate

test: # septa.py에 test 코드 설정이 되어 있다면 테스트를 진행한다.
  script:
    - python setup.py test
    - pip install tox flake8  # you can also use tox
    - tox -e py36,flake8

run: # dist에 .whl 파일을 패키지화 하여 생성한다.
  script:
    - python setup.py bdist_wheel # .whl 생성
    # an alternative approach is to install and run:
    - pip install dist/* # 그리고 설치한다
    # run the command here
  artifacts:
    paths:
      - dist/*.whl

pages: # 문서들을 웹 형태로 빌드 배포한다. 하지만 동작하진 않았다 make에서 실패..
  script:
    - pip install sphinx sphinx-rtd-theme
    - cd doc
    - make html
    - mv build/html/ ../public/
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

# pyscaffold를 적용한 gitlab ci/cd 사용
- to be update...
