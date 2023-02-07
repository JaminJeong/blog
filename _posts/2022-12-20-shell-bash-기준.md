---
title: "shell command"
date: 2022-12-22 00:00:00 -0000
categories: shell command ubuntu
---

# Linux 명령어 tips
* 자세한 내용은 --help, man page 혹은 인터넷 검색을 이용하자


# 접속

## ssh
### ssh-copy-id
* 접속할 때마다 키를 입력할 필요없이 키를 복사하여 저장함. 최초 한번 복사후 이후는 암호 없이 접속 가능함.
* ssh키 동기화 서비스를 이용하면 모든 서버에 암호입력 없이 접속 할 수 있도록 셋팅 할 수도 있음. 

```bash
$ ssh-copy-id jmjeong@nipa5.gpu.testdns.com
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/jamin/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
jmjeong@node5.gpu.testdns.dev's password: 

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'jmjeong@node5.gpu.testdns.dev'"
and check to make sure that only the key(s) you wanted were added.
```

* ~/.ssh/known_hosts에서 키가 추가된것을 확인 할 수 있음.

### ssh-keygen
* ssh 키를 생성함. ssh-copy-id와 gitlab에 ssh 키를 등록하기 위해 필요함.


```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jmjeong/.ssh/id_rsa): 
Created directory '/home/jmjeong/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/jmjeong/.ssh/id_rsa
Your public key has been saved in /home/jmjeong/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:dddd????????KKKKKKKKKKKKKKKKK jmjeong@testdns.dev@pathfinder
The key's randomart image is:
+---[RSA 3072]----+
| .Xo..           |
| o.B..           |
| +. + .          |
|=. . + + .       |
|+ + * + S        |
| * X = o         |
|E * * o          |
|=% o o           |
|B.o              |
+----[SHA256]-----+
```

* 해당 키 파일은 ~/.ssh/id_rsa와 ~/.ssh/id_rsa.pub에 생성됨.
* 해당 키 파일이 생성되는 것을 확인해야 함.
* 아래 부분에서 가급적 암호를 넣어서 사용해야 키를 분실 했을 때 안전함.

```bash
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
```


# 서버간 복사
* 가능하면 파일질라 같은 ui 툴을 사용하는게 편하지만 로컬 컴퓨터를 통하지 않고 복사하고 싶을 때가 많다.
## scp
* 기본 포맷은 user @ ip or dns : 복사할 곳의 폴더 혹은 파일
* -r 은 하위 폴더 이동

```bash
$ scp ./test.py user@node2.gpu.testdns.ai:/home/user 
$ scp -r ./data user@node2.gpu.testdns.ai:/home/user
$ scp -r user@node2.gpu.testdns.ai:/home/user/data ./data
```

## rsync
* 기본 포맷은 user @ ip or dns : 복사할 곳의 폴더 혹은 파일
* --delete 옵션은 Src에 없는 파일은 dst에서 삭제함.

```bash
// 로컬 데이터 백업
$ rsync -avzh /data /backup

// 로컬의 데이터를 리모트로 복제
$ rsync -avzh /data 192.168.1.100:/backup

// 사용자 계정을 지정하여 전송
$ rsync -avzh /data user@192.168.1.100:/backup

// ssh 포트가 다를 경우 지정
$ rsync -avzh -e 'ssh -p 8888' /data 192.168.1.100:/backup

// 백업 서버를 로컬로 복제
$ rsync -avzh 192.168.1.100:/backup /data
```

* reference : [rsync](https://luran.me/396)

# 서버간 파일 공유
* 기본적으로 nas로 모두 되기 때문에 특별한 경우가 아니면 사용하지 않는 것이 좋다

## sshfs
* 해당 명령어는 복사가 이동이 아니라 마운트이다.
* 로컬을 삭제 하면 원격도 삭제 된다.

```bash
$ # 연결이 끊기면 재접속할 수 있게 설정
$ sshfs -o reconnect user@x.x.x.x:/data /home/user/data
```

* reference : [sshfs](https://blog.leocat.kr/notes/2018/04/23/sshfs-mount-remote-server-using-ssh)

# 시스템 상태 보기

## htop
* top 명령어도 좋지만 비주얼적인 측면에서 더 유용하다.

```bash
$ htop
```

![SCR-20220516-mal](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/5482017697ea99572f0e7ce32d40995c/SCR-20220516-mal.png)

## nvidia-smi

```bash
$ nvidia-smi
```

![스크린샷_2022-05-16_오후_4.06.02](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/f27ea361b9512860d608c1f0f972053f/스크린샷_2022-05-16_오후_4.06.02.png)

## lshw
* 해당 머신의 하드웨어 스펙을 볼수 있다. 
* 가급적 less를 같이 사용하여 결과를 편하게 볼 수 있게 하자.

```bash
$ lshw | less
```


## watch
* 명령어를 1초에 한번 실행

```bash
$ watch -n 1 nvidia-smi
```

* watch -n 초 "명령어"
* ls -al 명령어를 0.5초 마다 실행

```bash
$ watch -n .5 "ls -al"
```


# 기타

## awk 
* 여러가지 동작을 시킬때 유용하다


```bash
# 특정 프로세스 삭제

## 삭제하고 싶은 프로세스 확인
$ ps aux | grep $USER | grep python
USER   PID   %CPU   %MEM   VSZ    RSS    TTY   STAT   START  TIME   COMMAND
root   1639   0.0   0.0.   37796  20943  ?     Ss     Apr29  0.01   /usr/bin/python3 train.py

## $2를 통해 두번째 출력 인자 즉 프로세스 아이디 추출
$ ps aux | grep $USER | grep python | awk '{print "kill -9 "$2 }'
kill -9 1639

## kill -9 1639 해당 명령어 실행
$ ps aux | grep $USER | grep python | awk '{print "kill -9 "$2 }' | bash 


# git 과 연계 delete 파일 복구
## 삭제하고 싶은 프로세스 확인
$ git status | grep delete
delete: setup.py

## $2를 통해 두번째 출력 인자 즉 setup.py 문자열 추출
$ git status | grep delete | awk '{print "git chekcout "$2 }'
git checkout setup.py

## git checkout setup.py 해당 명령어 실행
$ git status | grep delete | awk '{print "git chekcout "$2 }' | bash
```


## sed

## grep
* 특정 명령어 (cat, ls -al) 등의 출력이 나오는 명령어에서 라인 단위로 필요한 단어를 취합함
* -v 옵션을 이용하면 특정 단어를 포함한 열을 제외하고 출력
* -r 옵션을 이용하면 해당 폴더의 모든 파일에서 특정 단어를 포함한 문자열을 반환함


```bash
$ cat test.txt
testtest
hello world !!
testtest

$ cat test.txt | grep hello
hello world !!

$ cat test.txt | grep -v test
hello world !!

$ grep -r "hello" ./
```


## less
* 출력 결과나 파일 리스트를 vi의 커맨드 모드로 볼 수 있음
* 나올때는 q를 입력, / 를 통해 검색.

```bash
$ cat test.txt | less
testtest
hello world !!
testtest
```


## find
* 해당 파일이름을 해당 폴더에서 찾아서 위치를 알려줌

```bash
$ find ./ -name "hello.py"
```


## which
* 사용하고 있는 명령의 실행 파일 위치를 알려줌

```bash
$ witch python 
/opt/miniconda3/bin/python
```


## ps
* 컴퓨터 내부에서 돌고 있는 process 를 확인
* htop에서 확인할수 있는 프로세스를 명령어로 확인 할 수 있음.
* aux 옵션 a(자세히) u(user들만) x(실행되고 있는 상태만)
* grep 으로 $USER 즉 자신만 분리함 - echo $USER해 보면 자신의 아이디가 나옴

```bash
$ ps aux | grep $USER
$ ps aux | grep python
$ ps aux | grep $USER | less
```


## kill 
* 특정 프로세스를 중지 시킬 수 있음.

```bash
$ ps aux
USER   PID   %CPU   %MEM   VSZ    RSS    TTY   STAT   START  TIME   COMMAND
root   1639   0.0   0.0.   37796  20943  ?     Ss     Apr29  0.01   /usr/bin/python3 train.py

$ ps aux | grep python 
$ kill -9 1639
```


## history
* history 명령어를 써도 좋지만 가급적 ctrl + r 을 이용하여 이전 명령어 검색을 해서 이용하자
* [fzf](https://github.com/junegunn/fzf) 와 함께 ctrl + r 을 이용하면 더욱 편하다 
### 설치

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

### 사용
* ctrl + r

![스크린샷_2022-05-17_오전_10.38.52](https://raw.githubusercontent.com/JaminJeong/blog/main/_posts/uploads/8e933dfc887bb8d380ac74230b560a0a/스크린샷_2022-05-17_오전_10.38.52.png)

## tree

```bash
$ tree
.
├── README.md
├── image
│   ├── image-1.png
│   └── image.png
├── requirements.txt
├── sample
│   ├── README.md
│   ├── TODO-DATA.tsv
│   ├── TODO.tsv
│   └── teg_sample_data.xlsx
└── src
    ├── graph_analysis.py
    └── graph_analysis_test.ipynb

3 directories, 10 files 
```

## head
- 파일이 너무 커서 앞 부분만 보고 싶을 때 사용

```bash
## 기본 설정 시 맨 앞 5줄만 가져옴
$ head test.txt
## 정해준 숫자 만큼 가져옴
$ head -n 10 test.txt
$ head -10 test.txt
```

## tail
- 파일이 너무 커서 뒷 부분만 보고 싶을 때 사용
- 또는 로그를 보고 싶을 때 사용

```bash
## 기본 설정 시 맨 뒤 5줄만 가져옴
$ tail test.txt
## 정해준 숫자 만큼 가져옴
$ tail -10 test.txt
## 로그를 보고 싶을때 명령어가 종료 되지 않고 파일에 새로운 내용이 추가 되면 같이 보여줌.
$ tail -10f test.log
```


## background run
- 두가지 방법이 존재함.

```bash
## 1. 명령어 시작부터 background 실행. &를 붙여 줌. 
$ bash run.sh & 
## 2. 동작 하고 있는 중간에 ctrl + z 후에 bg 명령어
$ run.sh
....
ctrl+z
$ bg 
```

## foreground run
- background로 동작하던 것을 foreground로 가져옴.

```bash
## 명령어 시작부터 background 실행. &를 붙여 줌. 
$ bash run.sh & 
$ fg 
```


# ncdu
- 폴더 내부의 파일들의 용량을 쉽게 확인할 수 있고 파일 삭제 기능이 있어 용량 관리하기에 용이하다

```bash
$ ncdu 
```