---
title: "git"
date: 2022-12-22 00:00:00 -0000
categories: git 
---

# 목차
- [git의 설정 파일](#git-------)
- [global 설정](#global---)
  * [commit 을 위한 user 계정 설정](#commit------user------)
  * [에디터 설정](#------)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# git의 설정 파일
- 각각의 저장소 마다 설정 파일이 있다.
- 아래는 stylegan2의 예제이다.
```bash
$ cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = https://github.com/NVlabs/stylegan2
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```
- git 주소를 변경하고 싶거나 ssh 로 바꾸고 싶을 때 url 부분을 수정해 주면 된다
```bash 
 - url = https://github.com/NVlabs/stylegan2
 + url = git@github.com:NVlabs/stylegan2.git
```

# global 설정
- git의 global한 환경설정을 할수 있다.
- alias 기능을 이용하여 축약어를 쓸 수 있다. .bashrc에서 사용하는 alias와 동일하다
- log 관련 된 기능은 검색하여 찾음 것이다. 더 예쁜 것이 있으면 적용해도 좋다.
- user의 경우 명령어로 설정도 가능하지만 여기서 편집을 하여도 된다.
- user의 경우 저장소 마다 설정을 달리 할수 있다. 
  - 해당 저장소에서 .git/conifg 파일을 수정하면 된다.
- .gitconfig 파일을 각자 사용하는 서버에 $HOME폴더에 넣어 놓으면 해당 설정으로 모두 사용할 수 있다.
```bash
$ cat ~/.gitconfig
[alias]
	st = status
	s = status -s
	co = checkout
	ci = commit
	br = branch
	l = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
	pu = pull
	ch = "!git checkout $(git branch | fzf)"
	a = "!git add $(git status -s | fzf -m | awk '{print $2}')"
[user]
	name = user
	email = user@testdns.ai
```

## commit 을 위한 user 계정 설정
- [git 최초 설정](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EC%B5%9C%EC%B4%88-%EC%84%A4%EC%A0%95)을 참고하자
```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```
## 에디터 설정
```bash
$ git config --global core.editor vim
```


