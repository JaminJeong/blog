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