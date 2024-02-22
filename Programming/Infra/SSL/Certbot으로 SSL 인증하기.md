
## Snap을 사용하여 설치하기 (Recommended)

### Snap 설치

``` bash
sudo apt update
sudo apt install snapd    # snap 설치

sudo snap install hello-world    # 제대로 설치되었는지
hello-world 6.4 from Canonical✓ installed
hello-world
Hello World!
```

### 기존에 설치했던 Certbot 삭제

``` bash
sudo apt-get remove certbot
```

### Certbot 설치

``` bash
sudo snap install --classic certbot
```

### Cerbot 실행을 위한 심볼릭 링크 생성

``` bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Certbot 실행

``` bash
sudo certbot --nginx   # certbot이 nginx 설정을 자동으로 변경
sudo certbot certonly --nginx    # 사용자가 nginx 설정을 따로 해주어야 함
```

실행 후에는 안내하는 대로 커맨드를 입력하면 된다.
SSL 인증을 하고자 하는 도메인명을 입력하면 인증서가 설치된다.

### 인증서 자동 갱신 테스트

``` bash
sudo certbot renew --dry-run
```


### Trouble Shooting

##### Certbot 인증서 설치 오류

```bash
Deploying certificate
Could not install certificate

NEXT STEPS:
- The certificate was saved, but could not be installed (installer: nginx). After fixing the error shown below, try installing it again by running:
  certbot install --cert-name konggogi.store-0002

Could not automatically find a matching server block for konggogi.store. Set the `server_name` directive to use the Nginx installer.
Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile /var/log/letsencrypt/letsencrypt.log or re-run Certbot with -v for more details.
```

**원인**
- Nginx의 config 파일에 `server_name`으로 도메인명을 지정해주지 않았기 때문이다. (멍청이...)

**Certbot을 사용하기 위해 Nginx 설정을 해주어야 하는 이유**
1. Certbot은 Nginx config 파일을 찾는다.
2. 해당 파일의 server 블록에 작성된 `server_name`의 도메인들을 가져온다.
``` bash
Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: konggogi.store
2: dev.konggogi.store
3: prod.konggogi.store
4: www.konggogi.store
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 4
Requesting a certificate for www.konggogi.store
```

**해결**
[Let’s Encrypt 인증서로 NGINX SSL 설정하기](https://nginxstore.com/blog/nginx/lets-encrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C%EB%A1%9C-nginx-ssl-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)


### Ref

[Certbot Documentation](https://eff-certbot.readthedocs.io/en/latest/install.html#installation)
[Certbot Instructions](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal) - Nginx + Ubuntu 조합
