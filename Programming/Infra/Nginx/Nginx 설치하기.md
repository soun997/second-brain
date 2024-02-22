
### 필수 패키지 설치

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

### Nginx apt 저장소 공개키 다운로드

``` bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

### 유효한 공개키인지 검사

``` bash
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

**output**
``` bash
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

fingerprint가 `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62`과 같은지 확인
아니라면 유효한 공개키가 아니므로 다운로드한 파일을 삭제한다.

### Nginx apt 저장소 세팅 (Stable)

``` bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

### 패키지 저장소 우선순위 변경

``` bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

### Nginx 설치

``` bash
sudo apt update
sudo apt install nginx
```


![[Pasted image 20231125040608.png]]

성공!


### Trouble Shooting

##### 서비스 실행 에러 - `(98: Address already in use)`

**원인**
- Nginx는 80포트를 사용하는데, 누군가 80포트를 사용하고 있기 때문에 서비스 시작이 안됐다.
- 확인해보니 이미 Nginx가 사용하고 있었다..?

**해결**
- 80포트를 점유하고 있는 Nginx 프로세스를 종료하고 다시 서비스를 시작하니 정상작동 하였다.

##### 주소창에 ip를 입력해도 nginx 페이지가 나오지 않음, 접근 불가

**원인**
- EC2의 인바운드 규칙에 80포트를 허용해주지 않았기 때문이다.

**해결**
- 인바운드 규칙에 80포트를 추가해주었다.


### Ref
[Nginx Documentation](https://nginx.org/en/linux_packages.html#Ubuntu)
