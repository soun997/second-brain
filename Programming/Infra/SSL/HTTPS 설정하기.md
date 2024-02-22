
> AWS EC2, Ubuntu Jammy 22.04 (LTS), Nginx - [[Nginx 설치하기]]

### 도메인 구입

HTTPS 프로토콜을 사용하기 위해서는 도메인이 필요하다.

우리나라의 가장 유명한 도메인 공급 업체인 가비아에서 구매를 진행했다.
> https://www.gabia.com/


### 도메인명에 IP 주소 매핑

>여기서 헷갈릴 수 있는 부분이 ‘DNS 호스트’ 설정과 ‘DNS 정보’ 내의 ‘DNS 레코드 설정’인데 둘은 큰 차이가 있습니다. 가비아 고객센터에서도 [관련 내용의 문서](https://customer.gabia.com/faq/detail#/2929/2948)가 있습니다. DNS 호스트는 자체적으로 네임 서버를 구축하여 이용하기 위해 그 호스트를 등록하는 것입니다. 즉, 가비아 네임 서버를 사용하는 지금의 경우에는 필요한 설정이 아닙니다. 아까 nslookup을 사용했을 때 발생한 SERVFAIL을 해결하기 위해서, 그리고 현재의 목적인 도메인으로 IP 주소를 대체하기 위해서는 DNS 레코드(resource record)를 추가해야합니다. 따라서 ‘DNS 레코드 설정’으로 들어갑니다. ‘DNS 설정’에서 구매한 도메인의 설정으로 한 번 더 진입하면 다음과 같은 화면이 나타납니다.

가비아 사이트 - 도메인 관리 - DNS 정보 - DNS 관리 툴 페이지로 들어가서 매핑시켜준다.

**A타입**
조사 필요

**CNAME 타입**
조사 필요

**서브 도메인 지정**
조사 필요


![[Pasted image 20231125043820.png]]

> 도메인명과 IP 주소를 매핑한게 적용되기까지 꽤 시간이 걸리는 듯 하다...(2시간?)


### [[Certbot으로 SSL 인증하기]]

Certbot을 사용해 무료로 SSL 인증서를 발급받도록 하겠다.


### 성공!

![[Pasted image 20231125060055.png]]


### Trouble Shooting

##### SSL 인증 이후에 404 오류 발생

**원인**
- Certbot이 자동으로 생성해준 코드를 너무 믿었다.
- endpoint에 대한 페이지가 매핑이 되어있지 않아서 404 오류가 발생한 것이었다.

**해결**
- root endpoint로 요청이 들어올 경우 index 페이지를 보여주도록 했다.
```
server {
	...
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
