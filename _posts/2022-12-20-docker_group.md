---
title: "docker setting"
date: 2022-12-22 00:00:00 -0000
categories: docker
---

# docker 그룹 추가 

```bash
$ sudo groupadd docker 
```



# docker 유저 추가

```bash
$ sudo usermod -aG docker $USER
$ sudo service docker restart
```


# docker 유저 활성화

```bash
$ newgrp docker
```