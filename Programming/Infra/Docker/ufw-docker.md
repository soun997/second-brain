
## 설치 방법

### ufw-docker 스크립트를 다운로드한다.

``` bash
sudo wget -O /usr/local/bin/ufw-docker \
  https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
sudo chmod +x /usr/local/bin/ufw-docker
```

### ufw-docker를 설치한다.

``` bash
sudo ufw-docker install
sudo systemctl restart ufw
```

해당 명령어를 통해 UFW의 `after.rules`을 변경할 수 있다.


## 사용 방법

#### UFW 설정 확인

``` bash
sudo ufw-docker check
```

#### 허용된 포트 확인

``` bash
sudo ufw-docker status
```

#### 특정 컨테이너와 연관된 firewall rules

``` bash
sudo ufw-docker list {컨테이너명}
```

#### 특정 컨테이너의 port 개방

``` bash
sudo ufw-docker allow {컨테이너명} 80
```

``` bash
sudo ufw-docker allow {컨테이너명} 443/tcp
```

#### 특정 컨테이너의 port 폐쇄

``` bash
sudo ufw status numbered    # 번호 확인
sudo ufw delete {#번호}
```

```
sudo ufw-docker delete allow {컨테이너명} 443/tcp
```


## Ref

[ufw-docker](https://github.com/chaifeng/ufw-docker#install)